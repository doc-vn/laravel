# Laravel Cashier

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Stripe](#stripe-configuration)
    - [Braintree](#braintree-configuration)
    - [Cấu hình loại tiền](#currency-configuration)
- [Subscription](#subscriptions)
    - [Tạo Subscription](#creating-subscriptions)
    - [Kiểm tra trạng thái Subscription](#checking-subscription-status)
    - [Thay đổi Plan](#changing-plans)
    - [Subscription số lượng lớn](#subscription-quantity)
    - [Thuế của Subscription](#subscription-taxes)
    - [Huỷ Subscription](#cancelling-subscriptions)
    - [Resume Subscription](#resuming-subscriptions)
    - [Cập nhật thẻ Credit](#updating-credit-cards)
- [Subscription dành cho dùng thử](#subscription-trials)
    - [Khai báo trước thẻ credit](#with-credit-card-up-front)
    - [Khai báo thẻ credit sau](#without-credit-card-up-front)
- [Xử lý Stripe Webhook](#handling-stripe-webhooks)
    - [Định nghĩa xử lý Webhook Event](#defining-webhook-event-handlers)
    - [Subscription bị thất bại](#handling-failed-subscriptions)
- [Xử lý Braintree Webhooks](#handling-braintree-webhooks)
    - [Định nghĩa xử lý Webhook Event](#defining-braintree-webhook-event-handlers)
    - [Subscription bị thất bại](#handling-braintree-failed-subscriptions)
- [Phí](#single-charges)
- [Hoá đơn](#invoices)
    - [Tạo hoá đơn PDF](#generating-invoice-pdfs)

<a name="introduction"></a>
## Giới thiệu

Laravel Cashier cung cấp một interface dễ hiểu, rõ ràng cho các dịch vụ thanh toán subscription trực tuyến như [Stripe's](https://stripe.com) và [Braintree's](https://www.braintreepayments.com). Nó gần như đã xử lý tất cả các đoạn code mà bạn đang sợ viết mà có liên quan đến các phần thanh toán subscription. Ngoài quản lý subscription cơ bản, Cashier cũng có thể xử lý cả các phiếu giảm giá, chuyển đổi subscription, đăng ký "nhiều" subscription, thời hạn hủy bỏ và thậm chí là tạo các file hóa đơn PDF.

> {note} Nếu bạn chỉ muốn thực hiện các khoản phí "một lần" và không cung cấp chế độ subscription, thì bạn không nên sử dụng Cashier. Thay vào đó, hãy sử dụng trực tiếp SDK của Stripe và Braintree.

<a name="configuration"></a>
## Cấu hình

<a name="stripe-configuration"></a>
### Stripe

#### Composer

Đầu tiên, thêm package Cashier cho Stripe vào trong các library của bạn:

    composer require "laravel/cashier":"~7.0"

#### Database Migrations

Trước khi sử dụng Cashier, chúng ta cũng cần [chuẩn bị cơ sở dữ liệu](/docs/{{version}}/migrations). Chúng ta cần thêm một số cột vào bảng `users` của bạn và tạo thêm một bảng `subscriptions` mới để lưu trữ tất cả các subscription cho khách hàng của bạn:

    Schema::table('users', function ($table) {
        $table->string('stripe_id')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('stripe_id');
        $table->string('stripe_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

Khi migration đã được tạo xong, hãy chạy lệnh Artisan `migrate`.

#### Billable Model

Tiếp theo, thêm trait `Billable` vào định nghĩa model của bạn. Trait này sẽ cung cấp các phương thức khác nhau cho phép bạn thực hiện các tác vụ thanh toán phổ biến, chẳng hạn như tạo subscription, áp dụng phiếu giảm giá hoặc cập nhật thông tin thẻ tín dụng:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### API Keys

Cuối cùng, bạn sẽ cần cấu hình key Stripe trong file cấu hình `services.php` của bạn. Bạn có thể lấy các key API Stripe này từ trong bảng control panel ở bên phía Stripe:

    'stripe' => [
        'model'  => App\User::class,
        'key' => env('STRIPE_KEY'),
        'secret' => env('STRIPE_SECRET'),
    ],

<a name="braintree-configuration"></a>
### Braintree

#### Braintree Caveats

Đối với nhiều hành động, việc sử dụng Stripe hoặc Braintree cho các chức năng Cashier là như nhau. Cả hai dịch vụ đều cung cấp thanh toán subscription bằng thẻ tín dụng nhưng Braintree thì có hỗ trợ thêm thanh toán qua PayPal. Nhưng, Braintree cũng thiếu một số tính năng mà được Stripe hỗ trợ. Bạn nên đọc những điều sau để quyết định xem nên sử dụng Stripe hay Braintree:

<div class="content-list" markdown="1">
- Braintree hỗ trợ PayPal trong khi Stripe thì không.
- Braintree không hỗ trợ các phương thức `tăng` hoặc `giảm` các subscription. Đây là một hạn chế của Braintree, không phải là hạn chế của Cashier.
- Braintree không hỗ trợ giảm giá dựa trên tỷ lệ phần trăm. Đây là một hạn chế của Braintree, không phải là hạn chế của Cashier.
</div>

#### Composer

Đầu tiên, thêm package Cashier cho Braintree vào trong các library của bạn:

    composer require "laravel/cashier-braintree":"~2.0"

#### Plan Credit Coupon

Trước khi sử dụng Cashier cùng với Braintree, bạn sẽ cần định nghĩa một chiết khấu `plan-credit` trong bảng control panel ở bên phía Braintree. Khoản chiết khấu này sẽ được sử dụng để subscription theo một tỷ lệ phù hợp trong trường hợp khách hàng của bạn thay đổi từ thanh toán theo năm sang thanh toán theo tháng hoặc từ thanh toán theo tháng sang thanh toán theo năm.

Số tiền chiết khấu được cấu hình trong control panel ở bên phía Braintree có thể là bất kỳ giá trị nào mà bạn muốn, nhưng Cashier sẽ ghi đè số tiền đã được định nghĩa đó bằng số tiền tùy chỉnh của bạn mỗi khi bạn áp dụng phiếu giảm giá. Phiếu giảm giá này là cần thiết vì Braintree không hỗ trợ các subscription dựa theo tần suất subscription.

#### Database Migrations

Trước khi sử dụng Cashier, chúng ta cũng cần [chuẩn bị cơ sở dữ liệu](/docs/{{version}}/migrations). Chúng ta cần thêm một số cột vào bảng `users` của bạn và tạo thêm một bảng `subscriptions` mới để lưu trữ tất cả các subscription cho khách hàng của bạn:

    Schema::table('users', function ($table) {
        $table->string('braintree_id')->nullable();
        $table->string('paypal_email')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('braintree_id');
        $table->string('braintree_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

Khi migration đã được tạo xong, hãy chạy lệnh Artisan `migrate`.

#### Billable Model

Tiếp theo, hãy thêm trait `Billable` vào định nghĩa model của bạn:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### API Keys

Tiếp theo, bạn sẽ cần cấu hình các tùy chọn sau trong file `services.php` của bạn:

    'braintree' => [
        'model'  => App\User::class,
        'environment' => env('BRAINTREE_ENV'),
        'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
        'public_key' => env('BRAINTREE_PUBLIC_KEY'),
        'private_key' => env('BRAINTREE_PRIVATE_KEY'),
    ],

Sau đó, bạn nên thêm các lệnh gọi SDK Braintree sau vào phương thức `boot` của service provider `AppServiceProvider`:

    \Braintree_Configuration::environment(config('services.braintree.environment'));
    \Braintree_Configuration::merchantId(config('services.braintree.merchant_id'));
    \Braintree_Configuration::publicKey(config('services.braintree.public_key'));
    \Braintree_Configuration::privateKey(config('services.braintree.private_key'));

<a name="currency-configuration"></a>
### Cấu hình loại tiền

Đơn vị tiền mặc định của Cashier là Đô la Mỹ (USD). Bạn có thể thay đổi loại tiền mặc định này bằng cách gọi phương thức `Cashier::useCurrency` từ trong phương thức `boot` của một trong những service provider của bạn. Phương thức `useCurrency` chấp nhận hai tham số string: một là loại tiền và hai là ký hiệu của loại tiền đó:

    use Laravel\Cashier\Cashier;

    Cashier::useCurrency('eur', '€');

<a name="subscriptions"></a>
## Subscription

<a name="creating-subscriptions"></a>
### Tạo Subscription

Để tạo một subscription, trước tiên hãy lấy ra một instance của model billable của bạn, thường là một instance của `App\User`. Khi bạn đã lấy được instance của model, bạn có thể sử dụng phương thức `newSubscription` để tạo ra một subscription cho model:

    $user = User::find(1);

    $user->newSubscription('main', 'premium')->create($stripeToken);

Tham số đầu tiên được truyền cho phương thức `newSubscription` phải là tên của một subscription. Nếu application của bạn chỉ cung cấp một subscription duy nhất, bạn có thể gọi đây là `main` hoặc `primary`. Tham số thứ hai là plan của Stripe hoặc Braintree mà người dùng đang chọn subscription. Giá trị này phải tương ứng với identifier của plan ở bên phía Stripe hoặc bên phía Braintree.

Phương thức `create` chấp nhận một Stripe credit card hoặc một source token, phương thức này sẽ bắt đầu việc subscription cũng như việc cập nhật cơ sở dữ liệu của bạn với ID khách hàng và các thông tin thanh toán có liên quan khác.

#### Additional User Details

Nếu bạn muốn thêm chi tiết khách hàng, bạn có thể làm bằng cách truyền chúng làm tham số thứ hai cho phương thức `create`:

    $user->newSubscription('main', 'monthly')->create($stripeToken, [
        'email' => $email,
    ]);

Để tìm hiểu thêm về các field được hỗ trợ bởi Stripe hoặc Braintree, hãy xem [tài liệu về tạo khách hàng](https://stripe.com/docs/api#create_customer) của Stripe hoặc [tài liệu của Braintree](https://developers.braintreepayments.com/reference/request/customer/create/php) tương ứng.

#### Coupons

Nếu bạn muốn áp dụng phiếu giảm giá khi tạo subscription, bạn có thể sử dụng phương thức `withCoupon`:

    $user->newSubscription('main', 'monthly')
         ->withCoupon('code')
         ->create($stripeToken);

<a name="checking-subscription-status"></a>
### Kiểm tra trạng thái Subscription

Khi người dùng đã subscription vào application của bạn, bạn có thể dễ dàng kiểm tra trạng thái subscription của họ bằng nhiều phương thức thuận tiện khác nhau. Đầu tiên, phương thức `subscribed` sẽ trả về `true` nếu người dùng đó có active subscription, ngay cả khi subscription hiện tại đang trong thời gian dùng thử:

    if ($user->subscribed('main')) {
        //
    }

Phương thức `subscribed` cũng là một cách tuyệt vời cho một [route middleware](/docs/{{version}}/middleware), cho phép bạn lọc quyền truy cập vào các route hoặc các controller dựa trên trạng thái subscription của người dùng:

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // This user is not a paying customer...
            return redirect('billing');
        }

        return $next($request);
    }

Nếu bạn muốn xác định xem người dùng đó có còn trong thời gian dùng thử hay không, bạn có thể sử dụng phương thức `onTrial`. Phương thức này có thể hữu ích để hiển thị cảnh báo cho người dùng biết rằng họ vẫn đang trong thời gian dùng thử:

    if ($user->subscription('main')->onTrial()) {
        //
    }

Phương thức `subscribedToPlan` có thể được sử dụng để xác định xem người dùng có đăng ký gói dịch vụ đã cho hay không dựa vào ID của gói đó trong Stripe / Braintree. Trong ví dụ này, chúng ta sẽ xác định xem subscription `main` của người dùng có đăng ký gói `monthly` hay không:

    if ($user->subscribedToPlan('monthly', 'main')) {
        //
    }

#### Cancelled Subscription Status

Để xác định xem người dùng đã từng subscription, nhưng sau đó đã hủy, bạn có thể sử dụng phương thức `cancelled`:

    if ($user->subscription('main')->cancelled()) {
        //
    }

Bạn cũng có thể xác định xem người dùng đã hủy subscription của họ hay chưa, hay vẫn còn trong "thời gian có hiệu lực" cho đến khi subscription hết hạn. Ví dụ: nếu người dùng hủy subscription vào ngày 5 tháng 3 mà dự kiến ban đầu là sẽ hết hạn vào ngày 10 tháng 3, thì người dùng sẽ ở trong "thời gian có hiệu lực" của họ cho đến ngày 10 tháng 3. Lưu ý rằng phương thức `subscribed` vẫn trả về `true` trong thời gian này:

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="changing-plans"></a>
### Thay đổi Plan

Sau khi một người dùng đã subscription vào application của bạn, đôi khi họ có thể muốn thay đổi sang gói subscription mới. Để chuyển người dùng sang subscription mới, hãy truyền vào một định danh của gói mới vào phương thức `swap`:

    $user = App\User::find(1);

    $user->subscription('main')->swap('provider-plan-id');

Nếu người dùng đang trong thời gian dùng thử, thì thời gian dùng thử sẽ được duy trì. Ngoài ra, nếu có "nhiều" subscription tồn tại, thì những subscription đó cũng sẽ vẫn được duy trì.

Nếu bạn muốn thay đổi gói và hủy tất cả các gói dùng thử mà người dùng hiện đang sử dụng, bạn có thể sử dụng phương thức `skipTrial`:

    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');

<a name="subscription-quantity"></a>
### Subscription số lượng lớn

> {note} Subscription số lượng lớn chỉ được hỗ trợ cho phiên bản Stripe của Cashier. Braintree không có tính năng này.

Thỉnh thoảng subscription có thể bị ảnh hưởng bởi "số lượng". Ví dụ: application của bạn có thể tính phí $10 mỗi tháng **cho mỗi người dùng** trên mỗi tài khoản. Để dễ dàng tăng hoặc giảm số lượng subscription của bạn, hãy sử dụng các phương thức `incrementQuantity` hoặc `decrementQuantity`:

    $user = User::find(1);

    $user->subscription('main')->incrementQuantity();

    // Add five to the subscription's current quantity...
    $user->subscription('main')->incrementQuantity(5);

    $user->subscription('main')->decrementQuantity();

    // Subtract five to the subscription's current quantity...
    $user->subscription('main')->decrementQuantity(5);

Ngoài ra, bạn cũng có thể set một số lượng cụ thể bằng phương thức `updateQuantity`:

    $user->subscription('main')->updateQuantity(10);

Phương thức `noProrate` có thể được sử dụng để cập nhật số lượng của subscription mà không cần chia tỷ lệ phí:

    $user->subscription('main')->noProrate()->updateQuantity(10);

Để biết thêm thông tin về số lượng đăng ký, hãy tham khảo [tài liệu của Stripe](https://stripe.com/docs/subscriptions/quantities).

<a name="subscription-taxes"></a>
### Thuế của Subscription

Để khai báo tỷ lệ phần trăm thuế mà người dùng sẽ phải trả cho một subscription, hãy implement phương thức `taxPercentage` trên model billable của bạn và trả về một giá trị số từ 0 đến 100, không quá 2 chữ số thập phân.

    public function taxPercentage() {
        return 20;
    }

Phương thức `taxPercentage` cho phép bạn áp dụng thuế suất cho từng model, có thể có hữu ích cho người dùng trải rộng trên nhiều quốc gia và có các loại thuế suất khác nhau.

> {note} Phương thức `taxPercentage` chỉ áp dụng cho các loại subscription. Nếu bạn sử dụng Cashier để thực hiện các khoản tính phí "một lần", bạn sẽ cần khai báo thuế suất thủ công tại thời điểm đó.

<a name="cancelling-subscriptions"></a>
### Huỷ Subscription

Để hủy một subscription, hãy gọi phương thức `cancel` trên subscription của người dùng:

    $user->subscription('main')->cancel();

Khi một subscription bị hủy, Cashier sẽ tự động set cột `ends_at` trong cơ sở dữ liệu của bạn. Cột này được sử dụng để biết xem khi nào phương thức `subscribed` sẽ bắt đầu trả về `false`. Ví dụ: nếu khách hàng hủy subscription vào ngày 1 tháng 3, nhưng subscription không thể kết thúc cho đến khi hết ngày 5 tháng 3, thì phương thức `subscribed` vẫn sẽ tiếp tục trả về `true` cho đến ngày 5 tháng 3.

Bạn có thể biết những người dùng đã hủy subscription của họ nhưng vẫn đang trong "thời gian subscription có hiệu lực" bằng cách sử dụng phương thức `onGracePeriod`:

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

Nếu bạn muốn hủy subscription ngay lập tức, hãy gọi phương thức `cancelNow` trên subscription của người dùng:

    $user->subscription('main')->cancelNow();

<a name="resuming-subscriptions"></a>
### Resume Subscription

Nếu một người dùng đã hủy subscription của họ và bạn muốn resume tiếp subscription đó, hãy sử dụng phương thức `resume`. Để resume một subscription, người dùng vẫn **phải** đang trong thời gian subscription có hiệu lực:

    $user->subscription('main')->resume();

Nếu người dùng đã hủy subscription nhưng sau đó lại muốn resume tiếp subscription đó trước khi subscription hết hạn, họ sẽ không bị tính tiền ngay lập tức. Thay vào đó, subscription của họ sẽ được kích hoạt lại và họ sẽ thanh toán theo đúng chu kỳ thanh toán ban đầu của họ.
<a name="updating-credit-cards"></a>
### Cập nhật thẻ Credit

Phương thức `updateCard` có thể được sử dụng để cập nhật thông tin thẻ tín dụng cho khách hàng của bạn. Phương thức này chấp nhận một Stripe token và sẽ được gắn với một thẻ tín dụng mới làm thanh toán mặc định:

    $user->updateCard($stripeToken);

<a name="subscription-trials"></a>
## Subscription dành cho dùng thử

<a name="with-credit-card-up-front"></a>
### Khai báo trước thẻ credit

Nếu bạn muốn cung cấp thời gian dùng thử cho khách hàng của bạn trong khi vẫn muốn thu thập thông tin thanh toán của khách hàng, bạn nên sử dụng phương thức `trialDays` khi tạo subscription của bạn:

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')
                ->trialDays(10)
                ->create($stripeToken);

Phương thức này sẽ set ngày kết thúc của thời gian dùng thử vào trong bản ghi subscription trong cơ sở dữ liệu, và sẽ bảo với Stripe cũng như Braintree rằng là sẽ không tính phí khách hàng cho đến khi hết ngày dùng thử.

> {note} Nếu subscription của khách hàng không bị hủy trước ngày kết thúc dùng thử, họ sẽ bị tính phí ngay khi hết hạn dùng thử, vì vậy bạn nên chắc chắn là đã thông báo cho khách hàng biết về ngày kết thúc dùng thử của họ.

Bạn có thể xác định xem người dùng hiện tại có đang trong thời gian dùng thử hay không bằng cách sử dụng phương thức `onTrial` trên instance người dùng hoặc phương thức `onTrial` trên instance subscription. Hai ví dụ dưới đây có kết quả giống hệt nhau:

    if ($user->onTrial('main')) {
        //
    }

    if ($user->subscription('main')->onTrial()) {
        //
    }

<a name="without-credit-card-up-front"></a>
### Khai báo thẻ credit sau

Nếu bạn muốn cung cấp thời gian dùng thử mà không muốn thu thập thông tin thanh toán của người dùng, bạn có thể set cột `trial_ends_at` trong bản ghi của người dùng thành ngày kết thúc dùng thử mà bạn mong muốn. Điều này thường được thực hiện trong quá trình đăng ký người dùng:

    $user = User::create([
        // Populate other user properties...
        'trial_ends_at' => now()->addDays(10),
    ]);

> {note} Hãy chắc chắn là bạn đã thêm [date mutator](/docs/{{version}}/eloquent-mutators#date-mutators) cho cột `trial_ends_at` vào định nghĩa model của bạn.

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

    $user->newSubscription('main', 'monthly')->create($stripeToken);

<a name="handling-stripe-webhooks"></a>
## Xử lý Stripe Webhook

Cả Stripe và Braintree đều có thể thông báo cho application của bạn về một loạt các event thông qua webhooks. Để xử lý các webhook của Stripe, hãy định nghĩa một route trỏ đến controller xử lý webhook của Cashier. Controller này sẽ xử lý tất cả các incoming webhook request và gửi chúng đến phương thức controller thích hợp:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} Khi mà bạn đã đăng ký route xong, hãy đảm bảo là cấu hình URL webhook đúng với URL trong bảng cài đặt control panel ở bên phía Stripe.

Mặc định, controller này sẽ tự động xử lý hủy subscription khi mà có quá nhiều lần chi trả không thành công (được định nghĩa trong cài đặt Stripe của bạn); tuy nhiên, bạn sẽ sớm khám phá ra rằng bạn có thể extend controller này để xử lý bất kỳ event webhook nào mà bạn muốn trong phần ở dưới.

#### Webhooks & CSRF Protection

Vì các webhook của Stripe cần bỏ qua bước [bảo vệ CSRF](/docs/{{version}}/csrf) của Laravel, nên bạn hãy chắc chắn là đã khai báo URI của Stripe là một ngoại lệ trong middleware `VerifyCsrfToken` của bạn hoặc bạn có thể khai báo route này ra khỏi group middleware `web`:

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### Định nghĩa xử lý Webhook Event

Cashier sẽ tự động xử lý hủy subscription nếu như các lần chi trả không thành công, nhưng nếu bạn muốn thêm các event webhook Stripe mà bạn muốn tự xử lý, thì hãy extend controller Webhook. Tên phương thức của bạn phải tương ứng với quy ước của Cashier, cụ thể là, các phương thức sẽ cần được thêm tiền tố là `handle` vào tên của webhook Stripe mà bạn muốn xử lý, theo kiểu "camel case". Ví dụ: nếu bạn muốn xử lý webhook `invoice.payment_succeeded`, thì bạn cần thêm một phương thức `handleInvoicePaymentSucceeded` vào controller:

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle a Stripe webhook.
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }

<a name="handling-failed-subscriptions"></a>
### Subscription bị thất bại

Vậy, nếu thẻ tín dụng của khách hàng hết hạn thì sao? Đừng lo lắng - Cashier có chứa một controller Webhook có thể dễ dàng hủy subscription của khách hàng cho bạn. Như đã lưu ý ở trên, tất cả những gì bạn cần làm là trỏ một route đến một controller:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

Đó là tất cả! Các khoản thanh toán không thành công sẽ được kiểm soát và xử lý bởi controller. Controller này sẽ hủy subscription của khách hàng khi Stripe xác định rằng subscription không thành công (thông thường là sau ba lần thanh toán không thành công).

<a name="handling-braintree-webhooks"></a>
## Xử lý Braintree Webhooks

Cả Stripe và Braintree đều có thể thông báo cho application của bạn về nhiều loại event thông qua webhooks. Để xử lý các webhook của Braintree, hãy định nghĩa một route trỏ đến controller webhook của Cashier. Controller này sẽ xử lý tất cả các incoming webhook request và gửi chúng đến các phương thức controller thích hợp:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} Khi bạn đã đăng ký xong route của bạn, hãy đảm bảo là bạn đã cấu hình URL webhook trong bảng cài đặt bên phía Braintree của bạn.

Mặc định, controller này sẽ tự động xử lý hủy các subscription mà có quá nhiều lần chi trả không thành công (được xác định bởi cài đặt Braintree của bạn); tuy nhiên, bạn sẽ sớm khám phá ra rằng bạn có thể extend controller này để xử lý bất kỳ event webhook nào bạn muốn trong phần ở dưới.

#### Webhooks & CSRF Protection

Vì các webhook của Braintree cần bỏ qua bước [bảo vệ CSRF](/docs/{{version}}/csrf) của Laravel, nên bạn hãy chắc chắn là đã khai báo URI của Braintree là một ngoại lệ trong middleware `VerifyCsrfToken` của bạn hoặc bạn có thể khai báo route này ra khỏi group middleware `web`:

    protected $except = [
        'braintree/*',
    ];

<a name="defining-braintree-webhook-event-handlers"></a>
### Định nghĩa xử lý Webhook Event

Cashier sẽ tự động xử lý hủy subscription nếu như các lần chi trả không thành công, nhưng nếu bạn muốn thêm các event webhook Braintree mà bạn muốn tự xử lý, hãy extend controller Webhook này. Tên phương thức của bạn phải tương ứng với các quy ước của Cashier, cụ thể, các phương thức nên được thêm tiền tố là `handle` vào tên của webhook Braintree mà bạn muốn xử lý, phải theo kiểu "camel case". Ví dụ: nếu bạn muốn xử lý webhook `dispute_opened`, bạn nên thêm một phương thức `handleDisputeOpened` vào controller:

    <?php

    namespace App\Http\Controllers;

    use Braintree\WebhookNotification;
    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle a Braintree webhook.
         *
         * @param  WebhookNotification  $webhook
         * @return Response
         */
        public function handleDisputeOpened(WebhookNotification $notification)
        {
            // Handle The Event
        }
    }

<a name="handling-braintree-failed-subscriptions"></a>
### Subscription bị thất bại

Vậy, nếu thẻ tín dụng của khách hàng hết hạn thì sao? Đừng lo lắng - Cashier có chứa một controller Webhook có thể dễ dàng hủy subscription của khách hàng cho bạn. Như đã lưu ý ở trên, tất cả những gì bạn cần làm là trỏ một route đến controller:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

Đó là tất cả! Các khoản thanh toán không thành công sẽ được kiểm soát và xử lý bởi controller. Controller sẽ hủy subscription của khách hàng khi Braintree xác định rằng subscription không thành công (thông thường sau ba lần thanh toán không thành công). Đừng quên: bạn sẽ cần cấu hình URI webhook trong bảng cài đặt của bên phía Braintree.

<a name="single-charges"></a>
## Phí

### Simple Charge

> {note} Khi sử dụng Stripe, phương thức `charge` chấp nhận số tiền mà bạn muốn tính phí theo **loại tiền được set trong application của bạn**. Tuy nhiên, khi sử dụng Braintree, bạn nên chuyển toàn bộ số tiền đó sang đô la rồi truyền vào phương thức `charge`:

Nếu bạn muốn thực hiện một khoản tính phí "một lần" đối với các thẻ tín dụng của khách hàng đã subscription, bạn có thể sử dụng phương thức `charge` trên một instance model billable.

    // Stripe Accepts Charges In Cents...
    $user->charge(100);

    // Braintree Accepts Charges In Dollars...
    $user->charge(1);

Phương thức `charge` chấp nhận một mảng làm tham số thứ hai của nó, cho phép bạn truyền vào bất kỳ tùy chọn nào mà bạn muốn cho việc tạo phí của Stripe hoặc của Braintree. Tham khảo tài liệu của Stripe hoặc Braintree về các tùy chọn có sẵn cho bạn khi tạo phí:

    $user->charge(100, [
        'custom_option' => $value,
    ]);

Phương thức `charge` sẽ đưa ra một ngoại lệ nếu việc tính phí không thành công. Nếu tính phí thành công, thì một response của Stripe hoặc Braintree sẽ được trả về từ phương thức:

    try {
        $response = $user->charge(100);
    } catch (Exception $e) {
        //
    }

### Charge With Invoice

Thỉnh thoảng bạn có thể cần phải tạo tính phí một lần nhưng cũng cần tạo cả một hóa đơn cho khoản phí đó để bạn có thể cung cấp hóa đơn PDF đó cho khách hàng của bạn. Phương thức `invoiceFor` cho phép bạn làm điều đó. Ví dụ: hãy gửi hóa đơn "phí một lần" cho khách hàng của bạn là $5.00:

    // Stripe Accepts Charges In Cents...
    $user->invoiceFor('One Time Fee', 500);

    // Braintree Accepts Charges In Dollars...
    $user->invoiceFor('One Time Fee', 5);

Hóa đơn sẽ được tính ngay lập tức với thẻ tín dụng của người dùng. Phương thức `invoiceFor` cũng chấp nhận một mảng làm tham số thứ ba của nó, cho phép bạn truyền vào bất kỳ tùy chọn nào mà bạn muốn cho việc tạo phí của Stripe hoặc Braintree:

    $user->invoiceFor('One Time Fee', 500, [
        'custom-option' => $value,
    ]);

> {note} Phương thức `invoiceFor` sẽ tạo ra một hóa đơn Stripe sẽ thử lại sau các lần thanh toán không thành công. Nếu bạn không muốn hóa đơn thử lại sau các lần trả phí không thành công, bạn sẽ cần phải close chúng bằng API Stripe sau lần tính phí không thành công đầu tiên.

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
            'vendor'  => 'Your Company',
            'product' => 'Your Product',
        ]);
    });
