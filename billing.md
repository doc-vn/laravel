# Laravel Cashier (Stripe)

- [Giới thiệu](#introduction)
- [Cập nhật Cashier](#upgrading-cashier)
- [Cài đặt](#installation)
    - [Database Migrations](#database-migrations)
- [Cấu hình](#configuration)
    - [Billable Model](#billable-model)
    - [API Keys](#api-keys)
    - [Cấu hình loại tiền](#currency-configuration)
    - [Cấu hình thuế](#tax-configuration)
    - [Logging](#logging)
    - [Dùng custom model](#using-custom-models)
- [Customers](#customers)
    - [Lấy Customers](#retrieving-customers)
    - [Tạo Customers](#creating-customers)
    - [Cập nhật Customers](#updating-customers)
    - [Số dư](#balances)
    - [Tax IDs](#tax-ids)
    - [Đồng bộ dữ liệu khác hành với Stripe](#syncing-customer-data-with-stripe)
    - [Cổng thanh toán](#billing-portal)
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
    - [Thay đổi gói](#changing-prices)
    - [Subscription số lượng lớn](#subscription-quantity)
    - [Subscription với nhiều sản phẩm](#subscriptions-with-multiple-products)
    - [Nhiều giá cho subscription](#multiple-subscriptions)
    - [Thanh toán theo số liệu](#metered-billing)
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
    - [Kiểm tra Webhook Signatures](#verifying-webhook-signatures)
- [Phí](#single-charges)
    - [Tính phí một lần](#simple-charge)
    - [Tính phí với hoá đơn](#charge-with-invoice)
    - [Tạo Payment Intents](#creating-payment-intents)
    - [Hoàn trả](#refunding-charges)
- [Checkout](#checkout)
    - [Product Checkouts](#product-checkouts)
    - [Single Charge Checkouts](#single-charge-checkouts)
    - [Subscription Checkouts](#subscription-checkouts)
    - [Collecting Tax IDs](#collecting-tax-ids)
    - [Guest Checkouts](#guest-checkouts)
- [Hoá đơn](#invoices)
    - [Lấy hoá đơn](#retrieving-invoices)
    - [Hoá đơn tiếp theo](#upcoming-invoices)
    - [Xem trước hóa đơn đăng ký](#previewing-subscription-invoices)
    - [Tạo hoá đơn PDF](#generating-invoice-pdfs)
- [Xử lý lỗi thanh toán](#handling-failed-payments)
- [Strong Customer Authentication (SCA)](#strong-customer-authentication)
    - [Thanh toán yêu cầu xác nhận bổ sung](#payments-requiring-additional-confirmation)
    - [Thông báo thanh toán Off-session](#off-session-payment-notifications)
- [Stripe SDK](#stripe-sdk)
- [Testing](#testing)

<a name="introduction"></a>
## Giới thiệu

[Laravel Cashier Stripe](https://github.com/laravel/cashier-stripe) cung cấp một interface dễ hiểu, rõ ràng cho các dịch vụ thanh toán subscription trực tuyến như [Stripe's](https://stripe.com). Nó gần như đã xử lý tất cả các đoạn code mà bạn đang sợ viết mà có liên quan đến các phần thanh toán subscription. Ngoài quản lý subscription cơ bản, Cashier cũng có thể xử lý cả các phiếu giảm giá, chuyển đổi subscription, đăng ký "nhiều" subscription, thời hạn hủy bỏ và thậm chí là tạo các file hóa đơn PDF.

<a name="upgrading-cashier"></a>
## Cập nhật Cashier

Khi nâng cấp lên phiên bản mới của Cashier, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/cashier-stripe/blob/master/UPGRADE.md).

> **Warning**
> Để tránh các thay đổi nghiêm trọng, Cashier sẽ sử dụng một phiên bản API Stripe cố định. Cashier 14 sẽ sử dụng phiên bản API Stripe `2022-11-15`. Phiên bản API Stripe này sẽ được cập nhật thành các bản phát hành nhỏ để sử dụng các tính năng và cải tiến mới của Stripe.

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt package Cashier cho Stripe bằng trình quản lý package Composer:

```shell
composer require laravel/cashier
```

> **Warning**
> Để đảm bảo Cashier xử lý đúng tất cả các event của Stripe, hãy nhớ [thiết lập xử lý webhook của Cashier](#handling-stripe-webhooks).

<a name="database-migrations"></a>
### Database Migrations

Service provider của Cashier sẽ đăng ký thư mục migration database của chính nó, vì vậy hãy nhớ migration cơ sở dữ liệu của bạn sau khi cài đặt package. Việc migration Cashier sẽ thêm một số cột vào bảng `users` của bạn cũng như tạo một bảng `subscriptions` mới để chứa tất cả các đăng ký của khách hàng của bạn:

```shell
php artisan migrate
```

Nếu bạn cần ghi đè các migration đi kèm với Cashier, bạn có thể export chúng bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

Nếu bạn muốn ngăn việc migration của Cashier chạy, bạn có thể sử dụng phương thức `ignoreMigrations` được Cashier cung cấp. Thông thường, phương thức này nên được gọi trong phương thức `register` trong `AppServiceProvider` của bạn:

    use Laravel\Cashier\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::ignoreMigrations();
    }

> **Warning**
> Stripe khuyến cáo rằng bất kỳ cột nào được sử dụng để lưu trữ Stripe identifier phải phân biệt giữa chữ hoa và chữ thường. Do đó, bạn nên đảm bảo collation cho cột `stripe_id` sẽ được set là `utf8_bin` khi dùng MySQL. Thông tin thêm về điều này có thể được tìm thấy trong [tài liệu Stripe](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible).

<a name="configuration"></a>
## Cấu hình

<a name="billable-model"></a>
### Billable Model

Trước khi sử dụng Cashier, hãy thêm trait `Billable` vào định nghĩa billable model của bạn. Thông thường, đây sẽ là model `App\Models\User`. Trait này sẽ cung cấp các phương thức khác nhau cho phép bạn thực hiện các tác vụ thanh toán phổ biến, chẳng hạn như tạo subscription, áp dụng phiếu giảm giá hoặc cập nhật thông tin phương thức thanh toán:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Cashier sẽ giả định rằng billable model của bạn sẽ là class `App\Models\User` đi kèm với Laravel. Nếu bạn muốn thay đổi điều này, bạn có thể chỉ định một model khác thông qua phương thức `useCustomerModel`. Phương thức này thường được gọi trong phương thức `boot` của class `AppServiceProvider` của bạn:

    use App\Models\Cashier\User;
    use Laravel\Cashier\Cashier;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Cashier::useCustomerModel(User::class);
    }

> **Warning**
> Nếu bạn đang sử dụng model khác, khác với model `App\Models\User` do Laravel cung cấp, bạn sẽ cần export và thay đổi [migration](#installation) được cung cấp để khớp với tên bảng của model thay thế mà bạn muốn.

<a name="api-keys"></a>
### API Keys

Tiếp theo, bạn nên cấu hình key API của Stripe trong file `.env` của ứng dụng của bạn. Bạn có thể lấy khóa API Stripe của bạn từ bảng điều khiển của Stripe.

```ini
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
```

> **Warning**
> Bạn nên đảm bảo là biến môi trường `STRIPE_WEBHOOK_SECRET` đã được định nghĩa trong file `.env` trong ứng dụng của bạn, vì biến này sẽ được sử dụng để đảm bảo là các webhook đến từ Stripe thực sự đến từ Stripe.

<a name="currency-configuration"></a>
### Cấu hình loại tiền

Đơn vị tiền mặc định của Cashier là Đô la Mỹ (USD). Bạn có thể thay đổi loại tiền mặc định này bằng cách set biến môi trường `CASHIER_CURRENCY` trong file `.env` của ứng dụng của bạn:

```ini
CASHIER_CURRENCY=eur
```

Ngoài việc cấu hình đơn vị tiền tệ của Cashier, bạn cũng có thể chỉ định ngôn ngữ được sử dụng khi định dạng tiền tệ để hiển thị trong hóa đơn. Cashier sử dụng [class `NumberFormatter` của PHP](https://www.php.net/manual/en/class.numberformatter.php) để set ngôn ngữ tiền tệ:

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> **Warning**
> Để sử dụng các ngôn ngữ khác, khác với ngôn ngữ `en`, hãy đảm bảo là extension của PHP `ext-intl` đã được cài đặt và cấu hình trên server của bạn.

<a name="tax-configuration"></a>
### Cấu hình thuế

Nhờ [Stripe Tax](https://stripe.com/tax), bạn có thể tự động tính thuế cho tất cả hóa đơn do Stripe tạo. Bạn có thể kích hoạt chức năng tính thuế tự động bằng cách gọi phương thức `calculateTaxes` trong phương thức `boot` của class `App\Providers\AppServiceProvider` trong ứng dụng của bạn:

    use Laravel\Cashier\Cashier;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Cashier::calculateTaxes();
    }

Sau khi chức năng tính thuế đã được bật, mọi đăng ký mới và mọi hóa đơn một lần sẽ được tính thuế tự động.

Để chức năng này hoạt động bình thường, chi tiết thanh toán của khách hàng, chẳng hạn như tên, địa chỉ và mã số thuế của khách hàng, cần phải được đồng bộ với Stripe. Bạn có thể sử dụng các phương thức [đồng bộ hóa dữ liệu khách hàng](#syncing-customer-data-with-stripe) và [Tax ID](#tax-ids) do Cashier cung cấp để thực hiện việc này.

> **Warning**
> Không thể tính thuế cho [các khoản phí đơn lẻ](#single-charges) hoặc [các thanh toán một lần](#single-charge-checkouts).

<a name="logging"></a>
### Logging

Cashier cho phép bạn chỉ định channel log nào sẽ được sử dụng khi ghi log lỗi Stripe nghiêm trọng. Bạn có thể chỉ định channel log đó bằng cách định nghĩa biến môi trường `CASHIER_LOGGER` trong file `.env` của ứng dụng của bạn:

```ini
CASHIER_LOGGER=stack
```

Các exception được tạo ra bởi lệnh gọi API tới Stripe sẽ được ghi lại thông qua channel log mặc định của ứng dụng của bạn.

<a name="using-custom-models"></a>
### Dùng custom model

Bạn có thể thoải mái extend các model mà Cashier sử dụng bên trong bằng cách định nghĩa thêm model của riêng bạn và extend các model Cashier tương ứng:

    use Laravel\Cashier\Subscription as CashierSubscription;

    class Subscription extends CashierSubscription
    {
        // ...
    }

Sau khi đã định nghĩa model của bạn, bạn có thể hướng dẫn Cashier sử dụng model tùy chỉnh của bạn thông qua class `Laravel\Cashier\Cashier`. Thông thường, bạn nên thông báo cho Cashier về các model tùy chỉnh của bạn trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

    use App\Models\Cashier\Subscription;
    use App\Models\Cashier\SubscriptionItem;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Cashier::useSubscriptionModel(Subscription::class);
        Cashier::useSubscriptionItemModel(SubscriptionItem::class);
    }

<a name="customers"></a>
## Customers

<a name="retrieving-customers"></a>
### Lấy Customers

Bạn có thể lấy ra một khách hàng bằng ID Stripe của họ thông qua phương thức `Cashier::findBillable`. Phương thức này sẽ trả về một instance của billable model:

    use Laravel\Cashier\Cashier;

    $user = Cashier::findBillable($stripeId);

<a name="creating-customers"></a>
### Tạo Customers

Đôi khi, bạn có thể muốn tạo một Stripe customer mà không cần phải đăng ký. Bạn có thể thực hiện việc này bằng phương thức `createAsStripeCustomer`:

    $stripeCustomer = $user->createAsStripeCustomer();

Khi customer đã được tạo trong Stripe, bạn có thể bắt đầu subscription. Bạn có thể cung cấp mảng tùy chọn `$options` để truyền vào bất kỳ tham số [khách hàng nào được hỗ trợ bởi Stripe API](https://stripe.com/docs/api/customers/create):

    $stripeCustomer = $user->createAsStripeCustomer($options);

Bạn có thể sử dụng phương thức `asStripeCustomer` nếu bạn muốn trả về một đối tượng customer Stripe cho một billable model:

    $stripeCustomer = $user->asStripeCustomer();

Phương thức `createOrGetStripeCustomer` có thể được sử dụng nếu bạn muốn lấy Stripe customer cho một billable model nhất định nhưng không chắc chắn là liệu billable model đó đã là customer trong Stripe hay chưa. Phương thức này sẽ tạo ra một customer mới trong Stripe nếu customer đó chưa tồn tại:

    $stripeCustomer = $user->createOrGetStripeCustomer();

<a name="updating-customers"></a>
### Cập nhật Customers

Đôi khi, bạn có thể muốn cập nhật trực tiếp thông tin cho customer của Stripe. Bạn có thể thực hiện việc này bằng phương thức `updateStripeCustomer`. Phương thức này chấp nhận một loạt [các tùy chọn cập nhật của khách hàng được API Stripe hỗ trợ](https://stripe.com/docs/api/customers/update):

    $stripeCustomer = $user->updateStripeCustomer($options);

<a name="balances"></a>
### Số dư

Stripe cho phép bạn gửi vào hoặc rút ra từ "số dư" của khách hàng. Sau đó, số dư này sẽ được gửi hoặc rút theo mỗi hóa đơn mới. Để kiểm tra tổng số dư của khách hàng, bạn có thể sử dụng phương thức `balance` có sẵn trên billable model của bạn. Phương thức `balance` sẽ trả về một chuỗi được định dạng biểu thị số dư bằng đơn vị tiền tệ của khách hàng:

    $balance = $user->balance();

Để gửi vào số dư của khách hàng, bạn có thể cung cấp giá trị cho phương thức `creditBalance`. Nếu bạn muốn, bạn cũng có thể cung cấp một mô tả:

    $user->creditBalance(-500, 'Premium customer top-up.');

Cung cấp giá trị cho phương thức `debitBalance` sẽ tương ứng với rút số dư của khách hàng:

    $user->debitBalance(300, 'Bad usage penalty.');

Phương thức `applyBalance` sẽ tạo giao dịch số dư khách hàng mới cho khách hàng. Bạn có thể lấy các bản ghi giao dịch này bằng phương thức `balanceTransactions`, phương thức này có thể hữu ích để cung cấp nhật ký gửi và rút cho khách hàng xem:

    // Retrieve all transactions...
    $transactions = $user->balanceTransactions();

    foreach ($transactions as $transaction) {
        // Transaction amount...
        $amount = $transaction->amount(); // $2.31

        // Retrieve the related invoice when available...
        $invoice = $transaction->invoice();
    }

<a name="tax-ids"></a>
### Tax IDs

Cashier cũng cung cấp một cách dễ dàng để quản lý mã số thuế của khách hàng. Ví dụ: phương thức `taxIds` có thể được sử dụng để lấy ra tất cả các [ID thuế](https://stripe.com/docs/api/customer_tax_ids/object) được gán cho một khách hàng dưới dạng một collection:

    $taxIds = $user->taxIds();

Bạn cũng có thể lấy ra ID thuế cụ thể cho một khách hàng bằng identifier của khách hàng đó:

    $taxId = $user->findTaxId('txi_belgium');

Bạn có thể tạo ra một mã số thuế mới bằng cách cung cấp [loại](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type) và giá trị cho phương thức `createTaxId`:

    $taxId = $user->createTaxId('eu_vat', 'BE0123456789');

Phương thức `createTaxId` sẽ ngay lập tức thêm ID VAT vào tài khoản của khách hàng. [Việc xác minh ID VAT cũng được thực hiện bởi Stripe](https://stripe.com/docs/invoicing/customer/tax-ids#validation); tuy nhiên, đây là một quá trình bất đồng bộ. Bạn có thể được nhận được thông báo về các cập nhật xác minh này bằng cách đăng ký event webhook `customer.tax_id.updated` và kiểm tra [các tham số ID VAT `verification`](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification). Để biết thêm thông tin về cách xử lý webhook, vui lòng tham khảo [tài liệu về định nghĩa webhook handler](#handling-stripe-webhooks).

Bạn có thể xóa ID thuế bằng phương thức `deleteTaxId`:

    $user->deleteTaxId('txi_belgium');

<a name="syncing-customer-data-with-stripe"></a>
### Đồng bộ dữ liệu khác hành với Stripe

Thông thường, khi người dùng ứng dụng của bạn cập nhật tên, địa chỉ email hoặc thông tin khác thì cũng được lưu trữ bởi Stripe, bạn nên thông báo cho Stripe về các cập nhật đó. Bằng cách đó, bản sao thông tin của Stripe sẽ đồng bộ với ứng dụng của bạn.

Để tự động hóa việc này, bạn có thể định nghĩa một event listener trên billable model của bạn tương tác với event `updated` của model. Sau đó, trong event listener đó của bạn có thể gọi phương thức `syncStripeCustomerDetails` trên model:

    use function Illuminate\Events\queueable;

    /**
     * The "booted" method of the model.
     *
     * @return void
     */
    protected static function booted()
    {
        static::updated(queueable(function ($customer) {
            if ($customer->hasStripeId()) {
                $customer->syncStripeCustomerDetails();
            }
        }));
    }

Bây giờ, mỗi khi model khách hàng của bạn được cập nhật, thông tin của nó sẽ được đồng bộ với Stripe. Để thuận tiện, Cashier sẽ tự động đồng bộ thông tin khách hàng của bạn với Stripe khi tạo khách hàng.

Bạn có thể tùy chỉnh các cột được sử dụng để đồng bộ thông tin khách hàng với Stripe bằng cách ghi đè nhiều phương thức do Cashier cung cấp. Ví dụ: bạn có thể ghi đè phương thức `stripeName` để tùy chỉnh thuộc tính sẽ được coi là "tên" của khách hàng khi Cashier đồng bộ thông tin khách hàng với Stripe:

    /**
     * Get the customer name that should be synced to Stripe.
     *
     * @return string|null
     */
    public function stripeName()
    {
        return $this->company_name;
    }

Tương tự, bạn có thể ghi đè các phương thức `stripeEmail`, `stripePhone`, `stripeAddress` và `stripePreferredLocales`. Các phương thức này sẽ đồng bộ thông tin khách hàng với các tham số khách hàng tương ứng khi [cập nhật đối tượng khách hàng Stripe](https://stripe.com/docs/api/customers/update). Nếu bạn muốn kiểm soát hoàn toàn quy trình đồng hóa thông tin khách hàng này, bạn có thể ghi đè phương thức `syncStripeCustomerDetails`.

<a name="billing-portal"></a>
### Cổng thanh toán

Stripe cung cấp [một cách dễ dàng để thiết lập một cổng thanh toán](https://stripe.com/docs/billing/subscriptions/customer-portal) để customer của bạn có thể quản lý subscription, phương thức thanh toán và xem lại lịch sử thanh toán của họ. Bạn có thể chuyển hướng người dùng của bạn đến cổng thanh toán bằng cách gọi phương thức `redirectToBillingPortal` trên billable model từ controller hoặc route:

    use Illuminate\Http\Request;

    Route::get('/billing-portal', function (Request $request) {
        return $request->user()->redirectToBillingPortal();
    });

Mặc định, khi người dùng kết thúc việc quản lý subscription của họ, họ sẽ có thể quay lại route `home` của ứng dụng của bạn thông qua một link trong cổng thanh toán Stripe. Bạn có thể cung cấp URL tùy biến mà người dùng sẽ được quay lại bằng cách truyền URL làm tham số cho phương thức `redirectToBillingPortal`:

    use Illuminate\Http\Request;

    Route::get('/billing-portal', function (Request $request) {
        return $request->user()->redirectToBillingPortal(route('billing'));
    });

Nếu bạn muốn tạo URL cho cổng thanh toán mà không cần phải tạo response chuyển hướng HTTP, bạn có thể gọi phương thức `billingPortalUrl`:

    $url = $user->billingPortalUrl(route('billing'));

<a name="payment-methods"></a>
## Phương thức thanh toán

<a name="storing-payment-methods"></a>
### Lưu phương thức thanh toán

Để tạo một subscription hoặc thực hiện tính phí "một lần" với Stripe, bạn sẽ cần lưu phương thức thanh toán và lấy identifier của phương thức đó từ Stripe. Cách tiếp cận được sử dụng theo mục đích khác nhau dựa trên việc bạn sẽ sử dụng phương thức thanh toán này cho việc thanh toán các subscription hay các khoản phí một lần, vì vậy chúng ta sẽ xem xét cả hai ví dụ bên dưới.

<a name="payment-methods-for-subscriptions"></a>
#### Phương thức thanh toán cho Subscription

Khi lưu trữ thông tin thẻ tín dụng của khách hàng để đăng ký sử dụng trong tương lai, "Setup Intent" của Stripe API sẽ phải được sử dụng để thu thập thông tin chi tiết về phương thức thanh toán của khách hàng. "Setup Intent" cho biết ý định tính phí phương thức thanh toán của khách hàng. Trait `Billable` của Cashier có chứa một phương thức `createSetupIntent` để dễ dàng tạo một Setup Intent mới. Bạn nên gọi phương thức này từ route hoặc controller sẽ hiển thị form thu thập chi tiết phương thức thanh toán cho khách hàng của bạn:

    return view('update-payment-method', [
        'intent' => $user->createSetupIntent()
    ]);

Sau khi bạn đã tạo xong Setup Intent và truyền nó đến view, bạn nên gắn secret của intent đó vào trong element sẽ thu thập phương thức thanh toán. Ví dụ: hãy xem xét form "cập nhật phương thức thanh toán" này:

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    Update Payment Method
</button>
```

Tiếp theo, thư viện Stripe.js có thể sử dụng element đó để gắn các [Stripe Element](https://stripe.com/docs/stripe-js) cần thiết vào form và thu thập chi tiết thanh toán của khách hàng một cách an toàn:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Tiếp theo, thẻ có thể được xác minh và một "identifier cho phương thức thanh toán" an toàn có thể được lấy ra từ Stripe bằng cách sử dụng [phương thức `confirmCardSetup` của Stripe](https://stripe.com/docs/js/setup_intents/confirm_card_setup):

```js
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
```

Sau khi thẻ đã được Stripe xác minh, bạn có thể truyền kết quả identifier `setupIntent.payment_method` vào ứng dụng Laravel của bạn, nơi nó có thể được lưu với thông tin khách hàng. Phương thức thanh toán này có thể được [thêm vào như là một phương thức thanh toán mới](#adding-payment-methods) hoặc [được sử dụng để cập nhật một phương thức thanh toán mặc định](#updating-the-default-payment-method). Bạn cũng có thể sử dụng ngay identifier phương thức thanh toán này để [tạo ra một subscription mới](#creating-subscriptions).

> **Note**
> Nếu bạn muốn biết thêm thông tin về Setup Intent và cách thu thập chi tiết thanh toán của khách hàng, vui lòng [xem lại tài liệu tổng quan do Stripe cung cấp](https://stripe.com/docs/payments/save-and-reuse#php).

<a name="payment-methods-for-single-charges"></a>
#### Phương thức thanh toán cho phí

Tất nhiên, khi thực hiện một khoản tính phí một lần đối với một phương thức thanh toán của khách hàng, chúng ta sẽ chỉ cần sử dụng một identifier phương thức thanh toán trong một lần duy nhất. Do các giới hạn của Stripe, bạn không thể sử dụng phương thức thanh toán mặc định của khách hàng cho các khoản tính phí một lần. Bạn phải cho phép khách hàng nhập chi tiết phương thức thanh toán của họ bằng thư viện Stripe.js. Ví dụ, hãy xem xét form sau:

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    Process Payment
</button>
```

Sau khi định nghĩa một form như vậy, thư viện Stripe.js có thể sử dụng element đó để gắn các [Stripe Element](https://stripe.com/docs/stripe-js) cần thiết vào form và thu thập chi tiết thanh toán của khách hàng một cách an toàn:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Tiếp theo, thẻ có thể được xác minh và một "identifier cho phương thức thanh toán" an toàn có thể được lấy ra từ Stripe bằng cách sử dụng [phương thức `createPaymentMethod` của Stripe](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method):

```js
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
```

Nếu thẻ được xác minh thành công, bạn có thể truyền `paymentMethod.id` vào ứng dụng Laravel của bạn và xử lý tiếp [các khoản tính phí một lần](#simple-charge).

<a name="retrieving-payment-methods"></a>
### Lấy phương thức thanh toán

Phương thức `PaymentMethods` trên instance billable model sẽ trả về một collection các instance `Laravel\Cashier\PaymentMethod`:

    $paymentMethods = $user->paymentMethods();

Mặc định, phương thức này sẽ trả về các phương thức thanh toán thuộc loại `card`. Để lấy ra phương thức thanh toán thuộc loại khác, bạn có thể chuyển `type` làm tham số cho phương thức:

    $paymentMethods = $user->paymentMethods('sepa_debit');

Để lấy phương thức thanh toán mặc định của customer, phương thức `defaultPaymentMethod` có thể được sử dụng:

    $paymentMethod = $user->defaultPaymentMethod();

Bạn có thể lấy ra một phương thức thanh toán cụ thể mà được gắn với một billable model bằng cách sử dụng phương thức `findPaymentMethod`:

    $paymentMethod = $user->findPaymentMethod($paymentMethodId);

<a name="check-for-a-payment-method"></a>
### Xác định người dùng có phương thức thanh toán hay không

Để xác định xem một billable model có phương thức thanh toán mặc định được gắn với tài khoản của họ hay không, hãy gọi phương thức `hasDefaultPaymentMethod`:

    if ($user->hasDefaultPaymentMethod()) {
        //
    }

Bạn có thể sử dụng phương thức `hasPaymentMethod` để xác định xem billable model có ít nhất một phương thức thanh toán được gắn với tài khoản của họ hay không:

    if ($user->hasPaymentMethod()) {
        //
    }

Phương thức này sẽ xác định xem billable model có phương thức thanh toán thuộc loại `card` hay không. Để xác định xem một phương thức thanh toán thuộc loại khác có tồn tại cho model hay không, bạn có thể chuyển `type` làm tham số cho phương thức:

    if ($user->hasPaymentMethod('sepa_debit')) {
        //
    }

<a name="updating-the-default-payment-method"></a>
### Cập nhật phương thức thanh toán mặc định

Phương thức `updateDefaultPaymentMethod` có thể được sử dụng để cập nhật thông tin về phương thức thanh toán mặc định của khách hàng. Phương thức này chấp nhận một identifier phương thức thanh toán của Stripe và sẽ gắn phương thức thanh toán mới làm phương thức thanh toán hóa đơn mặc định:

    $user->updateDefaultPaymentMethod($paymentMethod);

Để đồng bộ thông tin phương thức thanh toán mặc định của bạn với thông tin phương thức thanh toán mặc định của khách hàng trong Stripe, bạn có thể sử dụng phương thức `updateDefaultPaymentMethodFromStripe`:

    $user->updateDefaultPaymentMethodFromStripe();

> **Warning**
> Phương thức thanh toán mặc định của khách hàng chỉ có thể được sử dụng để lập hóa đơn và tạo một subscription mới. Do những hạn chế áp đặt bởi Stripe, nó sẽ không thể được sử dụng cho các khoản tính phí một lần.

<a name="adding-payment-methods"></a>
### Thêm phương thức thanh toán

Để thêm một phương thức thanh toán mới, bạn có thể gọi phương thức `addPaymentMethod` trên một model billable model, và truyền identifier phương thức thanh toán:

    $user->addPaymentMethod($paymentMethod);

> **Note**
> Để tìm hiểu cách lấy identifier phương thức thanh toán, vui lòng xem lại [tài liệu lưu trữ phương thức thanh toán](#storing-payment-methods).

<a name="deleting-payment-methods"></a>
### Xoá phương thức thanh toán

Để xóa một phương thức thanh toán, bạn có thể gọi phương thức `delete` trên instance `Laravel\Cashier\PaymentMethod` mà bạn muốn xóa:

    $paymentMethod->delete();

Phương thức `deletePaymentMethod` sẽ xóa một phương thức thanh toán cụ thể ra khỏi billable model:

    $user->deletePaymentMethod('pm_visa');

Phương thức `deletePaymentMethods` sẽ xóa tất cả thông tin về phương thức thanh toán cho một billable model:

    $user->deletePaymentMethods();

Mặc định, phương thức này sẽ xóa tất cả các phương thức thanh toán thuộc loại `card`. Để xóa tất cả các phương thức thanh toán thuộc loại khác, bạn có thể chuyển `type` làm tham số cho phương thức:

    $user->deletePaymentMethods('sepa_debit');

> **Warning**
>  Nếu người dùng có một subscription đang hoạt động, ứng dụng của bạn không nên cho phép họ xóa phương thức thanh toán mặc định của họ.

<a name="subscriptions"></a>
## Subscriptions

Subscription cung cấp một cách để thiết lập thanh toán định kỳ cho khách hàng của bạn. Stripe subscription được quản lý bởi Cashier sẽ cung cấp và hỗ trợ cho nhiều giá subscription, số lượng subscription, bản dùng thử, v.v.

<a name="creating-subscriptions"></a>
### Tạo Subscription

Để tạo một subscription, trước tiên hãy lấy ra một instance billable model của bạn, thường là một instance của `App\Models\User`. Khi bạn đã lấy được instance của model, bạn có thể sử dụng phương thức `newSubscription` để tạo ra một subscription cho model:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription(
            'default', 'price_monthly'
        )->create($request->paymentMethodId);

        // ...
    });

Tham số đầu tiên được truyền đến phương thức `newSubscription` phải là tên internal của subscription. Nếu ứng dụng của bạn chỉ cung cấp một loại subscription, bạn có thể gọi nó là `default` hoặc `primary`. Tên subscription này chỉ dành cho việc sử dụng trong ứng dụng và không nhằm mục đích hiển thị cho người dùng. Ngoài ra, nó không được chứa khoảng trắng và nó cũng không được thay đổi sau khi tạo subscription. Tham số thứ hai là giá cụ thể mà người dùng đang subscription. Giá trị này phải tương ứng với identifier của giá trong Stripe.

Phương thức `create` sẽ chấp nhận [một identifier phương thức thanh toán Stripe](#storing-payment-methods) hoặc một đối tượng `PaymentMethod` của Stripe, phương thức này sẽ bắt đầu đăng ký cũng như cập nhật cơ sở dữ liệu của bạn với ID khách hàng Stripe của billable model và các thông tin liên quan khác.

> **Warning**
> Việc truyền trực tiếp identifier phương thức thanh toán cho phương thức đăng ký `create` cũng sẽ tự động thêm phương thức đó vào phương thức thanh toán mà đang được lưu của người dùng.

<a name="collecting-recurring-payments-via-invoice-emails"></a>
#### Collecting Recurring Payments Via Invoice Emails

Thay vì tự động thu các khoản thanh toán định kỳ của khách hàng, bạn có thể hướng dẫn Stripe gửi hóa đơn qua email cho khách hàng mỗi khi khoản thanh toán định kỳ của khách hàng đến hạn. Sau đó, khách hàng có thể thanh toán hóa đơn theo cách thủ công khi khách hàng nhận được hóa đơn. Khách hàng sẽ không cần phải cung cấp phương thức thanh toán khi nhận các khoản thanh toán định kỳ qua hóa đơn:

    $user->newSubscription('default', 'price_monthly')->createAndSendInvoice();

Về thời gian khách hàng phải thanh toán hóa đơn trước khi gói đăng ký của họ bị hủy được xác định tùy chọn `days_until_due`. Mặc định, thời gian đó sẽ là 30 ngày; tuy nhiên, bạn có thể cung cấp một giá trị cụ thể cho tùy chọn này nếu muốn:

    $user->newSubscription('default', 'price_monthly')->createAndSendInvoice([], [
        'days_until_due' => 30
    ]);

<a name="subscription-quantities"></a>
#### Quantities

Nếu muốn set [số lượng](https://stripe.com/docs/billing/subscriptions/quantities) cụ thể cho gói subscription, bạn nên gọi phương thức `quantity` trên subscription builder trước khi tạo subscription:

    $user->newSubscription('default', 'price_monthly')
         ->quantity(5)
         ->create($paymentMethod);

<a name="additional-details"></a>
#### Additional Details

Nếu bạn muốn thêm chi tiết về các tùy chọn [customer](https://stripe.com/docs/api/customers/create) hoặc [subscription](https://stripe.com/docs/api/subscriptions/create) được hỗ trợ bởi Stripe, bạn có thể làm bằng cách truyền chúng làm tham số thứ hai và tham số thứ ba cho phương thức `create`:

    $user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
        'email' => $email,
    ], [
        'metadata' => ['note' => 'Some extra information.'],
    ]);

<a name="coupons"></a>
#### Coupons

Nếu bạn muốn áp dụng phiếu giảm giá khi tạo subscription, bạn có thể sử dụng phương thức `withCoupon`:

    $user->newSubscription('default', 'price_monthly')
         ->withCoupon('code')
         ->create($paymentMethod);

Hoặc, nếu bạn muốn áp dụng [mã khuyến mại Stripe](https://stripe.com/docs/billing/subscriptions/discounts/codes), bạn có thể sử dụng phương thức `withPromotionCode`:

    $user->newSubscription('default', 'price_monthly')
         ->withPromotionCode('promo_code_id')
         ->create($paymentMethod);

ID mã khuyến mại đã cho phải là ID của API Stripe được gán cho mã khuyến mại chứ không phải mã khuyến mại dành cho khách hàng. Nếu bạn cần tìm ID mã khuyến mãi dựa theo mã khuyến mãi mà khách hàng nhập vào, bạn có thể sử dụng phương thức `findPromotionCode`:

    // Find a promotion code ID by its customer facing code...
    $promotionCode = $user->findPromotionCode('SUMMERSALE');

    // Find an active promotion code ID by its customer facing code...
    $promotionCode = $user->findActivePromotionCode('SUMMERSALE');

Trong ví dụ trên, đối tượng `$promotionCode` được trả về là một instance của `Laravel\Cashier\PromotionCode`. Class này có một đối tượng `Stripe\PromotionCode` bên dưới. Bạn có thể lấy ra phiếu giảm giá liên quan đến mã khuyến mãi này bằng cách gọi phương thức `coupon`:

    $coupon = $user->findPromotionCode('SUMMERSALE')->coupon();

Instance phiếu giảm giá này cho phép bạn xác định xem số tiền được giảm là bao nhiêu và phiếu giảm giá này là phiếu giảm giá theo mức cố định hay là giảm giá dựa theo tỷ lệ phần trăm:

    if ($coupon->isPercentage()) {
        return $coupon->percentOff().'%'; // 21.5%
    } else {
        return $coupon->amountOff(); // $5.99
    }

Bạn cũng có thể lấy ra các khoản giảm giá hiện đang được áp dụng cho khách hàng hoặc subscription:

    $discount = $billable->discount();

    $discount = $subscription->discount();

Các instance `Laravel\Cashier\Discount` được trả về có một instance đối tượng `Stripe\Discount` bên dưới. Bạn có thể lấy ra phiếu giảm giá liên quan đến đợt giảm giá này bằng cách gọi phương thức `coupon`:

    $coupon = $subscription->discount()->coupon();

Nếu bạn muốn áp dụng một phiếu giảm giá hoặc một mã khuyến mãi mới cho một khách hàng hoặc một đăng ký, bạn có thể thực hiện việc này thông qua các phương thức `applyCoupon` hoặc `applyPromotionCode`:

    $billable->applyCoupon('coupon_id');
    $billable->applyPromotionCode('promotion_code_id');

    $subscription->applyCoupon('coupon_id');
    $subscription->applyPromotionCode('promotion_code_id');

Hãy nhớ rằng bạn nên sử dụng ID của API Stripe được gán cho mã khuyến mãi chứ không phải mã khuyến mãi mà khách hàng nhập vào. Chỉ có thể áp dụng một phiếu giảm giá hoặc mã khuyến mãi cho một khách hàng hoặc một subscription tại một thời điểm nhất định.

Để biết thêm thông tin về chủ đề này, vui lòng tham khảo tài liệu của Stripe về [phiếu giảm giá](https://stripe.com/docs/billing/subscriptions/coupons) và [mã khuyến mãi](https://stripe.com/docs/billing/subscriptions/coupons/codes).

<a name="adding-subscriptions"></a>
#### Adding Subscriptions

Nếu bạn muốn thêm một subscription cho một khách hàng đã có sẵn phương thức thanh toán mặc định, bạn có thể gọi phương thức `add` trên subscription builder:

    use App\Models\User;

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')->add();

<a name="creating-subscriptions-from-the-stripe-dashboard"></a>
#### Creating Subscriptions From The Stripe Dashboard

Bạn cũng có thể tạo đăng ký từ chính bảng điều khiển trong Stripe. Khi làm như vậy, Cashier sẽ đồng bộ các đăng ký mới được thêm vào và gán cho chúng một tên `default`. Để tùy chỉnh tên đăng ký mà được gán cho các đăng ký đã tạo trên bảng điều khiển, [extend `WebhookController`](#defining-webhook-event-handlers) và ghi đè phương thức `newSubscriptionName`.

Ngoài ra, bạn chỉ có thể tạo một loại đăng ký thông qua bảng điều khiển Stripe. Nếu ứng dụng của bạn cung cấp nhiều đăng ký dùng nhiều tên khác nhau, thì cũng chỉ có thể thêm một loại đăng ký thông qua bảng điều khiển Stripe.

Cuối cùng, bạn phải luôn đảm bảo chỉ thêm một active đăng ký cho mỗi loại đăng ký do ứng dụng của bạn cung cấp. Nếu một khách hàng có hai đăng ký `default`, thì chỉ có một đăng ký được thêm gần đây nhất mới được Cashier sử dụng mặc dù cả hai sẽ được đồng bộ với cơ sở dữ liệu của ứng dụng của bạn.

<a name="checking-subscription-status"></a>
### Kiểm tra trạng thái Subscription

Khi customer đã subscription vào application của bạn, bạn có thể dễ dàng kiểm tra trạng thái subscription của họ bằng nhiều phương thức thuận tiện khác nhau. Đầu tiên, phương thức `subscribed` sẽ trả về `true` nếu customer đó có active subscription, ngay cả khi subscription hiện tại đang trong thời gian dùng thử. Phương thức `subscribed` chấp nhận tên của subscription làm tham số đầu tiên của nó:

    if ($user->subscribed('default')) {
        //
    }

Phương thức `subscribed` cũng là một cách tuyệt vời cho một [route middleware](/docs/{{version}}/middleware), cho phép bạn lọc quyền truy cập vào các route hoặc các controller dựa trên trạng thái subscription của người dùng:

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

Phương thức `subscribedToProduct` có thể được sử dụng để xác định xem người dùng có đăng ký một sản phẩm nhất định hay không dựa vào identifier của sản phẩm Stripe đó. Trong Stripe, các sản phẩm là tập hợp giá. Trong ví dụ này, chúng ta sẽ xác định xem đăng ký `default` của người dùng đang active đăng ký sản phẩm "premium" trong ứng dụng của chúng ta hay không. Identifier sản phẩm Stripe nhất định phải tương ứng với một trong các identifier sản phẩm có trong bảng điều khiển Stripe:

    if ($user->subscribedToProduct('prod_premium', 'default')) {
        //
    }

Bằng cách truyền một mảng cho phương thức `subscribedToProduct`, bạn có thể xác định xem đăng ký `default` của người dùng có active đăng ký các sản phẩm "basic" hay "premium" của ứng dụng hay không:

    if ($user->subscribedToProduct(['prod_basic', 'prod_premium'], 'default')) {
        //
    }

Phương thức `subscribedToPrice` có thể được sử dụng để xác định xem đăng ký của khách hàng có tương ứng với một ID giá hay không:

    if ($user->subscribedToPrice('price_basic_monthly', 'default')) {
        //
    }

Phương thức `recurring` có thể được sử dụng để xác định xem người dùng hiện tại có đang đăng ký và không còn trong thời gian dùng thử hay không:

    if ($user->subscription('default')->recurring()) {
        //
    }

> **Warning**
> Nếu người dùng có hai subscription cùng tên, thì subscription gần đây nhất sẽ luôn được trả về bằng phương thức `subscription`. Ví dụ: một người dùng có thể có hai record subscription có tên `default`; tuy nhiên, một trong các subscription có thể là subscription cũ và đã hết hạn, trong khi subscription còn lại là subscription hiện tại và đang hoạt động. Subscription gần đây nhất sẽ luôn được trả lại trong khi các subscription cũ hơn sẽ được lưu trong cơ sở dữ liệu để xem lại lịch sử.

<a name="cancelled-subscription-status"></a>
#### Canceled Subscription Status

Để xác định xem người dùng đã từng có subscription nhưng đã bị hủy đăng ký đó hay không, bạn có thể sử dụng phương thức `canceled`:

    if ($user->subscription('default')->canceled()) {
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

Tương tự, nếu hành động thanh toán phụ được yêu cầu khi hoán đổi prices subscription sẽ được đánh dấu là `past_due`. Khi subscription của bạn ở một trong hai trạng thái này, subscription sẽ không được active cho đến khi khách hàng xác nhận thanh toán. Để xác định xem subscription có được thanh toán hay chưa, bạn có thể thực hiện bằng cách sử dụng phương thức `hasIncompletePayment` trên billable model hoặc một instance subscription:

    if ($user->hasIncompletePayment('default')) {
        //
    }

    if ($user->subscription('default')->hasIncompletePayment()) {
        //
    }

Khi một subscription có một khoản thanh toán chưa hoàn thành, bạn nên hướng người dùng đến trang xác nhận thanh toán của Cashier, và truyền identifier của `latestPayment`. Bạn có thể sử dụng phương thức `latestPayment` có sẵn trên instance subscription để lấy identifier này:

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    Please confirm your payment.
</a>
```

Nếu bạn muốn một subscription vẫn được coi là hoạt động khi nó ở trạng thái `past_due` hoặc trạng thái `incomplete`, bạn có thể sử dụng phương thức `keepPastDueSubscriptionsActive` và phương thức `keepIncompleteSubscriptionsActive` do Cashier cung cấp. Thông thường, phương thức này nên được gọi trong phương thức `register` trong `App\Providers\AppServiceProvider` của bạn:

    use Laravel\Cashier\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::keepPastDueSubscriptionsActive();
        Cashier::keepIncompleteSubscriptionsActive();
    }

> **Warning**
> Khi một subscription ở trạng thái `incomplete`, bạn sẽ không thể thay đổi subscription cho đến khi xác nhận thanh toán. Do đó, các phương thức `swap` và `updateQuantity` sẽ đưa ra một ngoại lệ khi subscription ở trạng thái `incomplete`.

<a name="subscription-scopes"></a>
#### Subscription Scopes

Hầu hết các trạng thái subscription đều có sẵn dưới dạng query scope để bạn có thể dễ dàng truy vấn cơ sở dữ liệu của bạn để lấy ra các subscription có trạng thái nhất định:

    // Get all active subscriptions...
    $subscriptions = Subscription::query()->active()->get();

    // Get all of the canceled subscriptions for a user...
    $subscriptions = $user->subscriptions()->canceled()->get();

Dưới đây là danh sách đầy đủ các scope khả dụng:

    Subscription::query()->active();
    Subscription::query()->canceled();
    Subscription::query()->ended();
    Subscription::query()->incomplete();
    Subscription::query()->notCanceled();
    Subscription::query()->notOnGracePeriod();
    Subscription::query()->notOnTrial();
    Subscription::query()->onGracePeriod();
    Subscription::query()->onTrial();
    Subscription::query()->pastDue();
    Subscription::query()->recurring();

<a name="changing-prices"></a>
### Thay đổi gói

Sau khi khách hàng đăng ký ứng dụng của bạn, đôi khi họ có thể muốn thay đổi sang một subscription mới. Để hoán đổi một khách hàng sang một mức giá mới, hãy truyền identifier của giá Stripe cho phương thức `swap`. Khi hoán đổi giá, nó sẽ giả định rằng người dùng muốn kích hoạt lại subscription của họ nếu nó đã bị hủy trước đó. Identifier giá nhất định phải tương ứng với identifier giá Stripe có sẵn trong bảng điều khiển Stripe:

    use App\Models\User;

    $user = App\Models\User::find(1);

    $user->subscription('default')->swap('price_yearly');

Nếu customer đang trong thời gian dùng thử, thì thời gian dùng thử sẽ được duy trì. Ngoài ra, nếu có "nhiều" subscription tồn tại, thì những subscription đó cũng sẽ vẫn được duy trì.

Nếu bạn muốn thay đổi prices và hủy tất cả các price dùng thử mà customer hiện đang sử dụng, bạn có thể gọi phương thức `skipTrial`:

    $user->subscription('default')
            ->skipTrial()
            ->swap('price_yearly');

Nếu bạn muốn thay đổi prices và lập hóa đơn ngay cho customer thay vì đợi đến chu kỳ thanh toán tiếp theo của họ, bạn có thể sử dụng phương pháp `swapAndInvoice`:

    $user = User::find(1);

    $user->subscription('default')->swapAndInvoice('price_yearly');

<a name="prorations"></a>
#### Prorations

Mặc định, Stripe sẽ tính phí khi hoán đổi giữa các prices. Phương thức `noProrate` có thể được sử dụng để cập nhật price của subscription mà không bị tính phí:

    $user->subscription('default')->noProrate()->swap('price_yearly');

Để biết thêm thông tin về tính phí subscription, hãy tham khảo [tài liệu Stripe](https://stripe.com/docs/billing/subscriptions/prorations).

> **Warning**
> Việc thực thi phương thức `noProrate` trước phương thức `swapAndInvoice` sẽ không ảnh hưởng gì đến tỷ lệ. Một hóa đơn sẽ luôn được phát hành.

<a name="subscription-quantity"></a>
### Subscription số lượng lớn

Thỉnh thoảng subscription có thể bị ảnh hưởng bởi "số lượng". Ví dụ: một ứng dụng quản lý dự án có thể tính phí $10 mỗi tháng cho mỗi dự án. Bạn có thể sử dụng các phương thức `incrementQuantity` và `decrementQuantity` để dễ dàng tăng hoặc giảm số lượng đăng ký của bạn:

    use App\Models\User;

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

Để biết thêm thông tin về số lượng đăng ký, hãy tham khảo [tài liệu của Stripe](https://stripe.com/docs/subscriptions/quantities).

<a name="quantities-for-subscription-with-multiple-products"></a>
#### Quantities For Subscriptions With Multiple Products

Nếu subscription của bạn là [subscription nhiều sản phẩm](#subscriptions-with-multiple-products), bạn nên truyền tên của giá cùng số lượng mà bạn muốn tăng hoặc giảm làm tham số thứ hai cho các phương thức tăng hoặc giảm:

    $user->subscription('default')->incrementQuantity(1, 'price_chat');

<a name="subscriptions-with-multiple-products"></a>
### Subscription với nhiều sản phẩm

[Subscription nhiều sản phẩm](https://stripe.com/docs/billing/subscriptions/multiple-products) cho phép bạn chỉ định nhiều loại sản phẩm thanh toán cho cùng một đăng ký. Ví dụ: hãy tưởng tượng bạn đang xây dựng một ứng dụng "hỗ trợ" chăm sóc khách hàng có giá đăng ký cơ bản là 10 đô la mỗi tháng nhưng có thêm chức năng trò chuyện trực tiếp với mức giá là 15 đô la mỗi tháng. Thông tin về subscription nhiều sản phẩm sẽ được lưu trữ trong bảng cơ sở dữ liệu `subscription_items` của Cashier.

Bạn có thể chỉ định nhiều loại sản phẩm cho một gói subscription nhất định bằng cách truyền vào một mảng giá làm tham số thứ hai cho phương thức `newSubscription`:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default', [
            'price_monthly',
            'price_chat',
        ])->create($request->paymentMethodId);

        // ...
    });

Trong ví dụ trên, khách hàng sẽ có hai mức giá được đính kèm với một subscription `default`. Cả hai mức giá sẽ được tính theo khoảng thời gian thanh toán tương ứng. Nếu cần, bạn có thể sử dụng thêm phương thức `quantity` để chỉ ra số lượng cụ thể cho từng mức giá:

    $user = User::find(1);

    $user->newSubscription('default', ['price_monthly', 'price_chat'])
        ->quantity(5, 'price_chat')
        ->create($paymentMethod);

Nếu bạn muốn thêm một mức giá khác vào trong một subscription có sẵn, bạn có thể gọi phương thức `addPrice` của subscription:

    $user = User::find(1);

    $user->subscription('default')->addPrice('price_chat');

Ví dụ trên sẽ thêm price mới và khách hàng sẽ phải thanh toán cho price đó vào chu kỳ thanh toán tiếp theo của họ. Nếu bạn muốn lập hóa đơn ngay cho khách hàng, bạn có thể sử dụng phương thức `addPriceAndInvoice`:

    $user->subscription('default')->addPriceAndInvoice('price_chat');

Nếu bạn muốn thêm một price với một số lượng cụ thể, bạn có thể truyền số lượng đó làm tham số thứ hai của phương thức `addPrice` hoặc `addPriceAndInvoice`:

    $user = User::find(1);

    $user->subscription('default')->addPrice('price_chat', 5);

Bạn có thể xóa giá ra khỏi subscription bằng phương thức `removePrice`:

    $user->subscription('default')->removePrice('price_chat');

> **Warning**
> Bạn không thể xóa cái giá cuối cùng còn lại trông một subscription. Thay vào đó, bạn chỉ cần hủy subscription.

<a name="swapping-prices"></a>
#### Swapping Prices

Bạn cũng có thể thay đổi giá mà được đính kèm trong subscription nhiều sản phẩm. Ví dụ: hãy tưởng tượng một khách hàng có thể đăng ký `price_basic` với thêm một sản phẩm bổ sung `price_chat` và bạn muốn nâng cấp khách hàng từ `price_basic` lên giá `price_pro`:

    use App\Models\User;

    $user = User::find(1);

    $user->subscription('default')->swap(['price_pro', 'price_chat']);

Khi thực hiện ví dụ ở trên, subscription item với `price_basic` sẽ bị xóa và subscription item mà có `price_chat` sẽ vẫn được giữ nguyên. Ngoài ra, một subscription item mới cho `price_pro` sẽ được tạo ra.

Bạn cũng có thể chỉ định thêm các tùy chọn cho subscription item bằng cách truyền vào một mảng gồm các cặp key và giá trị cho phương thức `swap`. Ví dụ: bạn có thể cần chỉ định thêm số lượng price subscription:

    $user = User::find(1);

    $user->subscription('default')->swap([
        'price_pro' => ['quantity' => 5],
        'price_chat'
    ]);

Nếu bạn muốn thay đổi một giá duy nhất trên một subscription, bạn có thể thực hiện việc này bằng cách sử dụng phương thức `swap` trên chính subscription item đó. Cách tiếp cận này đặc biệt hữu ích nếu bạn muốn giữ lại tất cả các dữ liệu hiện có trên các giá khác của subscription:

    $user = User::find(1);

    $user->subscription('default')
            ->findItemOrFail('price_basic')
            ->swap('price_pro');

<a name="proration"></a>
#### Proration

Mặc định, Stripe sẽ tính phí theo tỷ lệ khi thêm hoặc xóa price ra khỏi một subscription mà đăng ký nhiều sản phẩm. Nếu bạn muốn thực hiện điều chỉnh một price mà không cần theo tỷ lệ, bạn nên kết hợp thêm phương thức `noProrate` vào code thay đổi price của bạn:

    $user->subscription('default')->noProrate()->removePrice('price_chat');

<a name="swapping-quantities"></a>
#### Quantities

Nếu bạn muốn cập nhật số lượng price trên các subscription riêng lẻ, bạn có thể thực hiện việc này bằng cách sử dụng [phương thức quantity](#subscription-quantity) và truyền thêm tên của price đó làm tham số cho phương thức:

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity(5, 'price_chat');

    $user->subscription('default')->decrementQuantity(3, 'price_chat');

    $user->subscription('default')->updateQuantity(10, 'price_chat');

> **Warning**
> Khi một subscription có nhiều giá, thì các thuộc tính `stripe_price` và `quantity` trên model `Subscription` sẽ là `null`. Để truy cập vào một thuộc tính giá cụ thể, bạn nên sử dụng quan hệ `items` có sẵn trên model `Subscription`.

<a name="subscription-items"></a>
#### Subscription Items

Khi một subscription có nhiều price, nó sẽ có nhiều subscription "items" được lưu trữ trong bảng `subscription_items` trong cơ sở dữ liệu của bạn. Bạn có thể truy cập vào những thứ này thông qua quan hệ `items` trên subscription:

    use App\Models\User;

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->items->first();

    // Retrieve the Stripe price and quantity for a specific item...
    $stripePrice = $subscriptionItem->stripe_price;
    $quantity = $subscriptionItem->quantity;

Bạn cũng có thể lấy ra một price cụ thể bằng phương thức `findItemOrFail`:

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');

<a name="multiple-subscriptions"></a>
### Multiple Subscriptions

Stripe cho phép khách hàng của bạn có thể subscription nhiều loại cùng một lúc. Ví dụ: bạn có thể đang điều hành một phòng gym cung cấp các gói đăng ký bơi và các gói đăng ký tập thể dục và mỗi gói đăng ký lại có thể có các mức giá khác nhau. Tất nhiên, khách hàng có thể đăng ký một hoặc cả hai gói.

Khi ứng dụng của bạn tạo các đăng ký, bạn có thể cung cấp tên của đăng ký cho phương thức `newSubscription`. Tên có thể là bất kỳ chuỗi nào mà đại diện cho loại đăng ký mà người dùng đang muốn sử dụng:

    use Illuminate\Http\Request;

    Route::post('/swimming/subscribe', function (Request $request) {
        $request->user()->newSubscription('swimming')
            ->price('price_swimming_monthly')
            ->create($request->paymentMethodId);

        // ...
    });

Trong ví dụ trên, chúng ta đã đăng ký bơi hàng tháng cho khách hàng. Nhưng, sau này có thể họ muốn chuyển sang đăng ký theo dạng hàng năm. Khi điều chỉnh đăng ký của khách hàng, chúng ta có thể chỉ cần hoán đổi giá của đăng ký `swimming`:

    $user->subscription('swimming')->swap('price_swimming_yearly');

Tất nhiên, bạn cũng có thể hủy đăng ký:

    $user->subscription('swimming')->cancel();

<a name="metered-billing"></a>
### Thanh toán theo số liệu

[Thanh toán theo số liệu](https://stripe.com/docs/billing/subscriptions/metered-billing) cho phép bạn tính phí khách hàng dựa trên một mức sử dụng sản phẩm của bạn trong một chu kỳ thanh toán. Ví dụ: bạn có thể tính phí khách hàng dựa trên số lượng tin nhắn văn bản hoặc email mà họ đã gửi mỗi tháng.

Để bắt đầu sử dụng thanh toán theo số liệu, trước tiên bạn cần tạo ra một sản phẩm mới trong bảng điều khiển Stripe của bạn với giá đo lường. Sau đó, sử dụng `meteredPrice` để thêm ID của giá đo lường vào đăng ký của khách hàng:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default')
            ->meteredPrice('price_metered')
            ->create($request->paymentMethodId);

        // ...
    });

Bạn cũng có thể bắt đầu subscription theo đo lường thông qua [Stripe Checkout](#checkout):

    $checkout = Auth::user()
            ->newSubscription('default', [])
            ->meteredPrice('price_metered')
            ->checkout();

    return view('your-checkout-view', [
        'checkout' => $checkout,
    ]);

<a name="reporting-usage"></a>
#### Reporting Usage

Khi khách hàng của bạn sử dụng ứng dụng của bạn, bạn sẽ báo cáo việc sử dụng của họ cho Stripe biết để Stripe có thể được lập hóa đơn một cách chính xác. Để tăng mức sử dụng của subscription được đo, bạn có thể sử dụng phương thức `reportUsage`:

    $user = User::find(1);

    $user->subscription('default')->reportUsage();

Mặc định, "số lượng sử dụng" sẽ là 1 và được thêm vào trong thời hạn thanh toán. Ngoài ra, bạn có thể truyền thêm một lượng cụ thể "mức độ sử dụng" vào mức sử dụng của khách hàng trong thời hạn thanh toán:

    $user = User::find(1);

    $user->subscription('default')->reportUsage(15);

Nếu ứng dụng của bạn cho phép nhiều giá vào một đăng ký, thì bạn sẽ cần sử dụng phương thức `reportUsageFor` để chỉ định mức giá nào mà bạn muốn báo cáo mức độ sử dụng:

    $user = User::find(1);

    $user->subscription('default')->reportUsageFor('price_metered', 15);

Thỉnh thoảng, bạn có thể cần cập nhật mức sử dụng mà bạn đã báo cáo trước đó. Để thực hiện điều này, bạn có thể truyền một timestamp hoặc một instance của `DateTimeInterface` làm tham số thứ hai cho `reportUsage`. Khi làm như vậy, Stripe sẽ cập nhật mức độ sử dụng được báo cáo tại thời điểm đó. Bạn có thể tiếp tục cập nhật các record báo cáo sử dụng trước đó vì ngày và giờ vẫn ở trong thời hạn thanh toán hiện tại:

    $user = User::find(1);

    $user->subscription('default')->reportUsage(5, $timestamp);

<a name="retrieving-usage-records"></a>
#### Retrieving Usage Records

Để lấy ra các mức sử dụng trước đó của khách hàng, bạn có thể sử dụng phương thức `usageRecords` của một instance subscription:

    $user = User::find(1);

    $usageRecords = $user->subscription('default')->usageRecords();

Nếu ứng dụng của bạn cho phép nhiều giá vào một đăng ký, thì bạn có thể sử dụng phương thức `usageRecordsFor` để chỉ định mức giá đo lường nào mà bạn muốn lấy ra các record sử dụng:

    $user = User::find(1);

    $usageRecords = $user->subscription('default')->usageRecordsFor('price_metered');

Các phương thức `usageRecords` và `usageRecordsFor` sẽ trả về một instance Collection có chứa một mảng kết hợp của các record sử dụng. Bạn có thể lặp mảng này để hiển thị tổng mức sử dụng của khách hàng:

    @foreach ($usageRecords as $usageRecord)
        - Period Starting: {{ $usageRecord['period']['start'] }}
        - Period Ending: {{ $usageRecord['period']['end'] }}
        - Total Usage: {{ $usageRecord['total_usage'] }}
    @endforeach

Để tham khảo đầy đủ về tất cả dữ liệu sử dụng được trả về và cách sử dụng phân trang dựa trên con trỏ của Stripe, vui lòng tham khảo [tài liệu API chính thức của Stripe](https://stripe.com/docs/api/usage_records/subscription_item_summary_list).

<a name="subscription-taxes"></a>
### Thuế của Subscription

> **Warning**
> Thay vì tính Thuế theo cách thủ công, bạn có thể [tự động tính thuế bằng Stripe Tax](#tax-configuration)

Để khai báo thuế suất mà người dùng sẽ phải trả cho một subscription, bạn nên implement phương thức `taxRates` trên model billable của bạn và trả về một mảng chứa ID thuế suất Stripe. Bạn có thể định nghĩa các thuế suất này trong [bảng điều khiển Stripe của bạn](https://dashboard.stripe.com/test/tax-rates):

    /**
     * The tax rates that should apply to the customer's subscriptions.
     *
     * @return array
     */
    public function taxRates()
    {
        return ['txr_id'];
    }

Phương thức `taxRates` cho phép bạn áp dụng thuế suất trên từng customer, có thể hữu ích cho một người dùng trải dài trên nhiều quốc gia với nhiều loại thuế suất.

Nếu bạn đang cung cấp subscription với nhiều sản phẩm, bạn có thể định nghĩa các mức thuế suất khác nhau cho từng price bằng cách implement một phương thức `priceTaxRates` trên billable model của bạn:

    /**
     * The tax rates that should apply to the customer's subscriptions.
     *
     * @return array
     */
    public function priceTaxRates()
    {
        return [
            'price_monthly' => ['txr_id'],
        ];
    }

> **Warning**
> Phương thức `taxRates` chỉ áp dụng cho các loại phí subscription. Nếu bạn sử dụng Cashier để thực hiện các khoản tính phí "một lần", bạn sẽ cần phải chỉ định một loại thuế suất cụ thể tại thời điểm đó.

<a name="syncing-tax-rates"></a>
#### Syncing Tax Rates

Khi thay đổi hard-code ID thuế suất được trả về từ phương thức `taxRates`, thì cài đặt thuế có trên bất kỳ subscription nào hiện có cho người dùng vẫn sẽ được giữ nguyên. Nếu bạn muốn cập nhật giá trị thuế cho các subscription hiện có với các giá trị `taxRates` mới, bạn nên gọi phương thức `syncTaxRates` trên instance subscription của người dùng:

    $user->subscription('default')->syncTaxRates();

Điều này cũng sẽ đồng bộ bất kỳ thuế suất item nào có trong subscription với nhiều sản phẩm. Nếu ứng dụng của bạn đang cung cấp subscription với nhiều sản phẩm, thì bạn nên đảm bảo là billable model của bạn đã implement phương thức `priceTaxRates` như [đã thảo luận ở trên](#subscription-taxes).

<a name="tax-exemption"></a>
#### Tax Exemption

Cashier cũng cung cấp các phương thức `isNotTaxExempt`, `isTaxExempt` và `reverseChargeApplies` để xác định xem khách hàng có được miễn thuế hay không. Các phương thức này sẽ gọi Stripe API để xác định trạng thái miễn thuế của khách hàng:

    use App\Models\User;

    $user = User::find(1);

    $user->isTaxExempt();
    $user->isNotTaxExempt();
    $user->reverseChargeApplies();

> **Warning**
> Các phương thức này cũng có sẵn trên tất cả các đối tượng `Laravel\Cashier\Invoice`. Tuy nhiên, khi gọi đối tượng `Invoice`, thì các phương thức đó sẽ xác định trạng thái miễn trừ tại thời điểm mà tạo hóa đơn.

<a name="subscription-anchor-date"></a>
### Subscription cố định ngày

Mặc định, ngày cố định thanh toán là ngày đã tạo ra subscription hoặc nếu có thời gian dùng thử, thì ngày dùng thử kết thúc sẽ là ngày thanh toán. Nếu bạn muốn sửa ngày cố định thanh toán, bạn có thể sử dụng phương thức `anchorBillingCycleOn`:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $anchor = Carbon::parse('first day of next month');

        $request->user()->newSubscription('default', 'price_monthly')
                    ->anchorBillingCycleOn($anchor->startOfDay())
                    ->create($request->paymentMethodId);

        // ...
    });

Để biết thêm thông tin về cách quản lý chu kỳ thanh toán của subscription, hãy tham khảo [tài liệu về chu kỳ thanh toán của Stripe](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>
### Huỷ Subscription

Để hủy một subscription, hãy gọi phương thức `cancel` trên subscription của người dùng:

    $user->subscription('default')->cancel();

Khi một subscription bị hủy, Cashier sẽ tự động set cột `ends_at` trong bảng `subscriptions` cơ sở dữ liệu của bạn. Cột này được sử dụng để biết xem khi nào phương thức `subscribed` sẽ bắt đầu trả về `false`.

Ví dụ: nếu khách hàng hủy subscription vào ngày 1 tháng 3, nhưng subscription không thể kết thúc cho đến khi hết ngày 5 tháng 3, thì phương thức `subscribed` vẫn sẽ tiếp tục trả về `true` cho đến ngày 5 tháng 3. Điều này được thực hiện vì người dùng thường được phép tiếp tục sử dụng ứng dụng cho đến khi kết thúc chu kỳ thanh toán của họ.

Bạn có thể biết những người dùng đã hủy subscription của họ nhưng vẫn đang trong "thời gian subscription có hiệu lực" bằng cách sử dụng phương thức `onGracePeriod`:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Nếu bạn muốn hủy subscription ngay lập tức, hãy gọi phương thức `cancelNow` trên subscription của người dùng:

    $user->subscription('default')->cancelNow();

Nếu bạn muốn hủy đăng ký ngay lập tức và lập hóa đơn cho bất kỳ mục hóa đơn nào theo tỷ lệ sử dụng chưa được lập hóa đơn còn lại hoặc các mục hóa đơn theo tỷ lệ mới hoặc đang chờ xử lý, hãy gọi phương thức `cancelNowAndInvoice` trên subscription của người dùng:

    $user->subscription('default')->cancelNowAndInvoice();

Bạn cũng có thể chọn hủy đăng ký tại một thời điểm cụ thể:

    $user->subscription('default')->cancelAt(
        now()->addDays(10)
    );

<a name="resuming-subscriptions"></a>
### Resume Subscription

Nếu một khách hàng đã hủy đăng ký của họ và bạn muốn tiếp tục đăng ký đó, bạn có thể gọi phương thức `resume` trên subscription đó. Khách hàng vẫn phải ở trong "thời gian gia hạn" để tiếp tục đăng ký:

    $user->subscription('default')->resume();

Nếu customer đã hủy subscription nhưng sau đó lại muốn resume tiếp subscription đó trước khi subscription hết hạn, customer sẽ không bị tính tiền ngay lập tức. Thay vào đó, subscription của họ sẽ được kích hoạt lại và họ sẽ thanh toán theo đúng chu kỳ thanh toán ban đầu của họ.

<a name="subscription-trials"></a>
## Subscription dành cho dùng thử

<a name="with-payment-method-up-front"></a>
### Khai báo trước phương thức thanh toán

Nếu bạn muốn cung cấp thời gian dùng thử cho khách hàng của bạn trong khi vẫn muốn thu thập thông tin thanh toán của khách hàng, bạn nên sử dụng phương thức `trialDays` khi tạo subscription của bạn:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default', 'price_monthly')
                    ->trialDays(10)
                    ->create($request->paymentMethodId);

        // ...
    });

Phương thức này sẽ set ngày kết thúc của thời gian dùng thử vào trong bản ghi subscription trong cơ sở dữ liệu, và sẽ bảo với Stripee là sẽ không tính phí khách hàng cho đến khi hết ngày dùng thử. Khi sử dụng phương thức `trialDays`, Cashier sẽ ghi đè lên bất kỳ khoảng thời gian dùng thử nào được cấu hình cho price trong Stripe.

> **Warning**
> Nếu subscription của khách hàng không bị hủy trước ngày kết thúc dùng thử, họ sẽ bị tính phí ngay khi hết hạn dùng thử, vì vậy bạn nên chắc chắn là đã thông báo cho khách hàng biết về ngày kết thúc dùng thử của họ.

Phương thức `trialUntil` cho phép bạn cung cấp một instance `DateTime` để chỉ định khi nào thời gian dùng thử kết thúc:

    use Carbon\Carbon;

    $user->newSubscription('default', 'price_monthly')
                ->trialUntil(Carbon::now()->addDays(10))
                ->create($paymentMethod);

Bạn có thể xác định xem người dùng hiện tại có đang trong thời gian dùng thử hay không bằng cách sử dụng phương thức `onTrial` trên instance người dùng hoặc phương thức `onTrial` trên instance subscription. Hai ví dụ dưới đây có kết quả tương đương:

    if ($user->onTrial('default')) {
        //
    }

    if ($user->subscription('default')->onTrial()) {
        //
    }

Bạn có thể sử dụng phương thức `endTrial` để kết thúc ngay một bản dùng thử subscription:

    $user->subscription('default')->endTrial();

Để xác định xem bản dùng thử hiện tại đã hết hạn hay chưa, bạn có thể sử dụng phương thức `hasExpiredTrial`:

    if ($user->hasExpiredTrial('default')) {
        //
    }

    if ($user->subscription('default')->hasExpiredTrial()) {
        //
    }

<a name="defining-trial-days-in-stripe-cashier"></a>
#### Defining Trial Days In Stripe / Cashier

Bạn có thể chọn định nghĩa số ngày dùng thử nhận được khi đăng ký price của bạn trong bảng điều khiển Stripe hoặc luôn truyền chúng vào thông qua Cashier. Nếu bạn chọn định nghĩa ngày dùng thử cho price của bạn trong Stripe, thì bạn nên biết rằng các subscription mới, bao gồm cả subscription mới cho những khách hàng đã có subscription trước đó, sẽ luôn nhận được thời gian dùng thử trừ khi bạn gọi phương thức `skipTrial()`.

<a name="without-payment-method-up-front"></a>
### Khai báo phương thức thanh toán sau

Nếu bạn muốn cung cấp thời gian dùng thử mà không muốn thu thập thông tin thanh toán của người dùng, bạn có thể set cột `trial_ends_at` trong bản ghi của người dùng thành ngày kết thúc dùng thử mà bạn mong muốn. Điều này thường được thực hiện trong quá trình đăng ký người dùng:

    use App\Models\User;

    $user = User::create([
        // ...
        'trial_ends_at' => now()->addDays(10),
    ]);

> **Warning**
> Hãy thêm [date cast](/docs/{{version}}/eloquent-mutators##date-casting) cho thuộc tính `trial_ends_at` trong định nghĩa của billable model của bạn.

Cashier sẽ coi các loại dùng thử như thế này là "dùng thử đại trà", vì nó sẽ không được gắn với bất kỳ thông tin subscription nào. Phương thức `onTrial` trên instance billable model sẽ trả về `true` nếu ngày hiện tại không vượt quá giá trị của ngày `trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }

Khi bạn đã sẵn sàng tạo một subscription thực sự cho người dùng, bạn có thể sử dụng phương thức `newSubscription` như bình thường:

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')->create($paymentMethod);

Để lấy ra ngày kết thúc dùng thử của người dùng, bạn có thể sử dụng phương thức `trialEndsAt`. Phương thức này sẽ trả về một instancen Carbon date nếu người dùng đang dùng thử hoặc là `null` nếu họ không dùng thử. Bạn cũng có thể truyền một tham số tùy chọn tên subscription nếu bạn muốn biết ngày kết thúc dùng thử cho một subscription cụ thể khác, khác với subscription mặc định:

    if ($user->onTrial()) {
        $trialEndsAt = $user->trialEndsAt('main');
    }

Bạn cũng có thể sử dụng phương thức `onGenericTrial` nếu bạn muốn biết cụ thể là người dùng đang trong thời gian dùng thử "bình thường" và chưa tạo subscription thực tế:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

<a name="extending-trials"></a>
### Mở rộng thời gian dùng thử

Phương thức `extendTrial` cho phép bạn kéo dài thời gian dùng thử của một subscription sau khi subscription được tạo. Nếu bản dùng thử đã hết hạn và khách hàng đã được lập hóa đơn cho subscription, bạn vẫn có thể cung cấp cho họ thêm thời gian dùng thử. Thời gian sử dụng trong thời gian dùng thử sẽ được trừ vào hóa đơn tiếp theo của khách hàng.

    use App\Models\User;

    $subscription = User::find(1)->subscription('default');

    // End the trial 7 days from now...
    $subscription->extendTrial(
        now()->addDays(7)
    );

    // Add an additional 5 days to the trial...
    $subscription->extendTrial(
        $subscription->trial_ends_at->addDays(5)
    );

<a name="handling-stripe-webhooks"></a>
## Xử lý Stripe Webhooks

> **Note**
> Bạn có thể sử dụng [the Stripe CLI](https://stripe.com/docs/stripe-cli) để giúp kiểm tra webhook trong quá trình phát triển local.

Stripe có thể thông báo cho ứng dụng của bạn về nhiều loại event khác nhau thông qua webhooks. Mặc định, sẽ có một route sẽ trỏ đến controller webhook của Cashier mà được đăng ký tự động thông qua service provider của Cashier. Trong controller này sẽ xử lý tất cả các request webhook đến.

Mặc định, the Cashier webhook controller sẽ tự động xử lý việc hủy subscription khi có quá nhiều lần tính phí không thành công (được định nghĩa trong cài đặt Stripe của bạn), cập nhật khách hàng, xóa khách hàng, cập nhật subscription và thay đổi phương thức thanh toán; tuy nhiên, bạn sẽ sớm khám phá ra là bạn có thể extend controller này để xử lý bất kỳ event webhook nào bạn thích.

Để đảm bảo ứng dụng của bạn có thể xử lý Stripe webhook, hãy nhớ cấu hình URL webhook trong bảng điều khiển Stripe. Mặc định, webhook controller của Cashier sẽ phản hồi trên đường dẫn URL là `/stripe/webhook`. Danh sách đầy đủ của tất cả các webhook mà bạn nên bật trong bảng điều khiển Stripe sẽ là:

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `payment_method.automatically_updated`
- `invoice.payment_action_required`
- `invoice.payment_succeeded`

Để thuận tiện, Cashier đã chứa một Artisan command `cashier:webhook`. Lệnh này sẽ tạo ra một webhook trong Stripe để lắng nghe tất cả các event mà Cashier yêu cầu:

```shell
php artisan cashier:webhook
```

Mặc định, webhook được tạo ra sẽ trỏ đến URL được định nghĩa bởi biến môi trường `APP_URL` và route `cashier.webhook` đi kèm với Cashier. Bạn có thể thêm tùy chọn `--url` khi gõ lệnh này nếu bạn muốn sử dụng một URL khác:

```shell
php artisan cashier:webhook --url "https://example.com/stripe/webhook"
```

Webhook được tạo ra sẽ sử dụng phiên bản Stripe API tương thích với phiên bản Cashier của bạn. Nếu bạn muốn sử dụng một phiên bản Stripe khác, bạn có thể thêm tùy chọn `--api-version`:

```shell
php artisan cashier:webhook --api-version="2019-12-03"
```

Sau khi tạo, webhook sẽ hoạt động ngay lập tức. Nếu bạn muốn tạo webhook nhưng tắt nó cho đến khi bạn sẵn sàng, bạn có thể cung cấp tùy chọn `--disabled` khi gọi lệnh:

```shell
php artisan cashier:webhook --disabled
```

> **Warning**
> Hãy đảm bảo rằng bạn đã bảo vệ các request webhook Stripe bằng middleware [xác minh signature webhook](#verifying-webhook-signatures) đi kèm của Cashier.

<a name="webhooks-csrf-protection"></a>
#### Webhooks & CSRF Protection

Vì các webhook của Stripe cần bỏ qua bước [bảo vệ CSRF](/docs/{{version}}/csrf) của Laravel, nên bạn hãy chắc chắn là đã khai báo URI của Stripe là một ngoại lệ trong middleware `App\Http\Middleware\VerifyCsrfToken` của bạn hoặc bạn có thể khai báo route này ra khỏi group middleware `web`:

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### Định nghĩa xử lý event Webhook

Cashier sẽ tự động xử lý việc hủy đăng ký nếu như các khoản phí không thành công và các sự kiện webhook phổ biến khác của Stripe. Tuy nhiên, nếu bạn muốn xử lý thêm các sự kiện webhook khác, thì bạn có thể làm bằng cách lắng nghe các sự kiện sau do Cashier gửi đi:

- `Laravel\Cashier\Events\WebhookReceived`
- `Laravel\Cashier\Events\WebhookHandled`

Cả hai sự kiện này đều chứa toàn bộ payload của webhook Stripe. Ví dụ: nếu bạn muốn xử lý webhook `invoice.payment_succeeded`, thì bạn có thể đăng ký [listener](/docs/{{version}}/events#defining-listeners) xử lý sự kiện đó:

    <?php

    namespace App\Listeners;

    use Laravel\Cashier\Events\WebhookReceived;

    class StripeEventListener
    {
        /**
         * Handle received Stripe webhooks.
         *
         * @param  \Laravel\Cashier\Events\WebhookReceived  $event
         * @return void
         */
        public function handle(WebhookReceived $event)
        {
            if ($event->payload['type'] === 'invoice.payment_succeeded') {
                // Handle the incoming event...
            }
        }
    }

Khi listener của bạn đã được định nghĩa, bạn có thể đăng ký nó trong `EventServiceProvider` của ứng dụng:

    <?php

    namespace App\Providers;

    use App\Listeners\StripeEventListener;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    use Laravel\Cashier\Events\WebhookReceived;

    class EventServiceProvider extends ServiceProvider
    {
        protected $listen = [
            WebhookReceived::class => [
                StripeEventListener::class,
            ],
        ];
    }

<a name="verifying-webhook-signatures"></a>
### Kiểm tra Webhook Signatures

Để bảo mật webhook của bạn, bạn có thể sử dụng [webhook signature của Stripe](https://stripe.com/docs/webhooks/signatures). Để thuận tiện, Cashier đã tự động thêm một middleware để kiểm tra các request webhook Stripe đến application là hợp lệ.

Để bật kiểm tra webhook, hãy đảm bảo rằng biến môi trường `STRIPE_WEBHOOK_SECRET` được set trong file `.env` trong apllication của bạn. Webhook `secret` có thể được lấy ra từ trang tổng quan trong tài khoản Stripe của bạn.

<a name="single-charges"></a>
## Phí

<a name="simple-charge"></a>
### Tính phí một lần

Nếu bạn muốn tính phí một lần đối với khách hàng, bạn có thể sử dụng phương thức `charge` trên một instance billable model. Bạn sẽ cần [cung cấp identifier phương thức thanh toán](#payment-methods-for-single-charges) làm tham số thứ hai cho phương thức `charge`:

    use Illuminate\Http\Request;

    Route::post('/purchase', function (Request $request) {
        $stripeCharge = $request->user()->charge(
            100, $request->paymentMethodId
        );

        // ...
    });

Phương thức `charge` chấp nhận một mảng làm tham số thứ ba của nó, cho phép bạn truyền vào bất kỳ tùy chọn nào mà bạn muốn cho việc tạo phí của Stripe. Bạn có thể tìm thêm về các thông tin tùy chọn có sẵn khi tạo khoản phí trong [tài liệu Stripe](https://stripe.com/docs/api/charges/create):

    $user->charge(100, $paymentMethod, [
        'custom_option' => $value,
    ]);

Bạn cũng có thể sử dụng phương thức `charge` mà không cần có customer hoặc người dùng. Để thực hiện điều này, hãy gọi phương thức `charge` trên một instance mới của billable model trong ứng dụng của bạn:

    use App\Models\User;

    $stripeCharge = (new User)->charge(100, $paymentMethod);

Phương thức `charge` sẽ đưa ra một ngoại lệ nếu việc tính phí không thành công. Nếu tính phí thành công, thì một instance của `Laravel\Cashier\Payment` sẽ được trả về từ phương thức:

    try {
        $payment = $user->charge(100, $paymentMethod);
    } catch (Exception $e) {
        //
    }

> **Warning**
> Phương thức `charge` chấp nhận số tiền thanh toán được ghi theo loại đơn vị thấp nhất của loại tiền tệ mà trong ứng dụng của bạn sử dụng. Ví dụ: nếu khách hàng thanh toán bằng Đô la Mỹ thì số tiền thanh toán phải được ghi theo đơn vị xu.

<a name="charge-with-invoice"></a>
### Tính phí với hoá đơn

Thỉnh thoảng bạn có thể cần phải tạo tính phí một lần và cung cấp hóa đơn PDF cho khách hàng của bạn. Phương thức `invoicePrice` cho phép bạn làm điều đó. Ví dụ: hãy gửi hóa đơn cho khách hàng của bạn là năm chiếc áo phông mới:

    $user->invoicePrice('price_tshirt', 5);

Hóa đơn sẽ được tính ngay lập tức cho phương thức thanh toán mặc định của người dùng. Phương thức `invoicePrice` cũng chấp nhận một mảng làm tham số thứ ba của nó. Mảng này sẽ chứa các tùy chọn thanh toán cho các hàng được thanh toán. Tham số thứ tư được phương thức chấp nhận cũng là một mảng sẽ chấp nhận các tùy chọn thanh toán cho chính hóa đơn đó:

    $user->invoicePrice('price_tshirt', 5, [
        'discounts' => [
            ['coupon' => 'SUMMER21SALE']
        ],
    ], [
        'default_tax_rates' => ['txr_id'],
    ]);

Tương tự như `invoicePrice`, bạn có thể sử dụng phương thức `tabPrice` để tạo khoản phí một lần cho nhiều mặt hàng (tối đa là 250 mặt hàng trên mỗi hóa đơn) bằng cách thêm chúng vào "tab" của khách hàng rồi lập hóa đơn cho khách hàng đó. Ví dụ: chúng ta có thể lập hóa đơn cho khách hàng với năm cái áo sơ mi và hai cái cốc:

    $user->tabPrice('price_tshirt', 5);
    $user->tabPrice('price_mug', 2);
    $user->invoice();

Ngoài ra, bạn có thể sử dụng phương thức `invoiceFor` để tính phí "một lần" đối với phương thức thanh toán mặc định của khách hàng:

    $user->invoiceFor('One Time Fee', 500);

Mặc dù phương thức `invoiceFor` có sẵn để bạn sử dụng nhưng bạn nên sử dụng phương thức `invoicePrice` và phương thức `tabPrice` cùng với mức giá được xác định trước. Bằng cách đó, bạn sẽ có quyền truy cập vào các phân tích và dữ liệu tốt hơn trong bảng điều khiển Stripe của bạn về doanh số bán hàng trên từng sản phẩm.

> **Warning**
> Phương thức `invoice`, `invoicePrice` và `invoiceFor` sẽ tạo ra một hóa đơn Stripe sẽ thử lại sau các lần thanh toán không thành công. Nếu bạn không muốn hóa đơn thử lại sau các lần trả phí không thành công, bạn sẽ cần phải close chúng bằng API Stripe sau lần tính phí không thành công đầu tiên.

<a name="creating-payment-intents"></a>
### Tạo Payment Intents

Bạn có thể tạo một payment intent Stripe mới bằng cách gọi phương thức `pay` trên một instance billable model. Việc gọi phương thức này sẽ tạo ra một payment intent được bao trong một instance `Laravel\Cashier\Payment`:

    use Illuminate\Http\Request;

    Route::post('/pay', function (Request $request) {
        $payment = $request->user()->pay(
            $request->get('amount')
        );

        return $payment->client_secret;
    });

Sau khi tạo xong payment intent, bạn có thể trả về một client secret cho frontend của ứng dụng để người dùng có thể hoàn tất thanh toán trong trình duyệt của họ. Để đọc thêm về cách xây dựng toàn bộ luồng thanh toán bằng cách sử dụng  payment intent của Stripe, vui lòng tham khảo [tài liệu của Stripe](https://stripe.com/docs/payments/accept-a-payment?platform=web).

Khi sử dụng phương thức `pay`, các phương thức thanh toán mặc định được cho phép trong bảng điều khiển Stripe của bạn sẽ được hiển thị cho khách hàng. Ngoài ra, nếu bạn chỉ muốn cho phép sử dụng một số phương thức thanh toán nhất định, bạn có thể sử dụng phương thức `payWith`:

    use Illuminate\Http\Request;

    Route::post('/pay', function (Request $request) {
        $payment = $request->user()->payWith(
            $request->get('amount'), ['card', 'bancontact']
        );

        return $payment->client_secret;
    });

> **Warning**
> Phương thức `pay` và `payWith` chấp nhận số tiền thanh toán được ghi theo loại đơn vị thấp nhất của loại tiền tệ mà trong ứng dụng của bạn sử dụng. Ví dụ: nếu khách hàng thanh toán bằng Đô la Mỹ thì số tiền thanh toán phải được ghi theo đơn vị xu.

<a name="refunding-charges"></a>
### Hoàn trả

Nếu bạn cần hoàn trả một phí đã được thanh toán trong Stripe, bạn có thể sử dụng phương thức `refund`. Phương thức này chấp nhận một [ID của Payment Intent Stripe](#payment-methods-for-single-charges) làm tham số đầu tiên của nó:

    $payment = $user->charge(100, $paymentMethodId);

    $user->refund($payment->id);

<a name="invoices"></a>
## Hoá đơn

<a name="retrieving-invoices"></a>
### Lấy hoá đơn

Bạn có thể dễ dàng lấy ra một mảng các hóa đơn của một model billable bằng cách sử dụng phương thức `invoices`. Phương thức `invoices` trả về một collection các instance `Laravel\Cashier\Invoice`:

    $invoices = $user->invoices();

Nếu bạn muốn thêm các hóa đơn đang chờ xử lý vào trong kết quả, bạn có thể sử dụng phương thức `invoicesIncludingPending`:

    $invoices = $user->invoicesIncludingPending();

Bạn có thể sử dụng phương thức `findInvoice` để lấy ra một hóa đơn cụ thể bằng ID chính nó:

    $invoice = $user->findInvoice($invoiceId);

<a name="displaying-invoice-information"></a>
#### Displaying Invoice Information

Khi liệt kê hóa đơn cho khách hàng, bạn có thể sử dụng các phương thức của hóa đơn để hiển thị thông tin hóa đơn. Ví dụ: bạn có thể muốn liệt kê tất cả các hóa đơn có trong một bảng, cho phép người dùng dễ dàng tải xuống bất kỳ cái nào trong số chúng:

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="upcoming-invoices"></a>
### Hoá đơn tiếp theo

Để lấy ra các hóa đơn sắp tới cho khách hàng, bạn có thể sử dụng phương thức `upcomingInvoice`:

    $invoice = $user->upcomingInvoice();

Tương tự, nếu khách hàng có nhiều subscription, bạn cũng có thể lấy ra hóa đơn sắp tới cho một subscription cụ thể:

    $invoice = $user->subscription('default')->upcomingInvoice();

<a name="previewing-subscription-invoices"></a>
### Xem trước hóa đơn đăng ký

Sử dụng phương thức `previewInvoice`, bạn có thể xem trước các hóa đơn trước khi thực hiện thay đổi giá. Điều này sẽ cho phép bạn xác định xem hóa đơn của khách hàng sẽ như thế nào khi thay đổi giá được thực hiện:

    $invoice = $user->subscription('default')->previewInvoice('price_yearly');

Bạn có thể truyền một mảng giá cho phương thức `previewInvoice` để xem trước nhiều hóa đơn với nhiều mức giá khác nhau:

    $invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);

<a name="generating-invoice-pdfs"></a>
### Tạo hoá đơn PDF

Trước khi tạo file hóa đơn PDF, bạn nên sử dụng Composer để cài đặt thư viện Dompdf, đây là thư viện tạo hóa đơn pdf mặc định của Cashier:

```php
composer require dompdf/dompdf
```

Từ trong một route hoặc một controller, bạn có thể sử dụng phương thức `downloadInvoice` để tạo một bản PDF cho hóa đơn đã cho để khách hàng có thể tải xuống. Phương thức này sẽ tự động tạo ra một response HTTP cần thiết để download gửi file hoá đơn:

    use Illuminate\Http\Request;

    Route::get('/user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId);
    });

Mặc định, tất cả dữ liệu trên hóa đơn được lấy từ thông tin khách hàng và dữ liệu hóa đơn này được lưu trữ trong Stripe. Tên file dựa trên giá trị được cấu hình trong `app.name` của bạn. Tuy nhiên, bạn có thể tùy chỉnh một số dữ liệu này bằng cách cung cấp một mảng làm tham số thứ hai cho phương thức `downloadInvoice`. Mảng này cho phép bạn tùy chỉnh thông tin như chi tiết về công ty và sản phẩm của bạn:

    return $request->user()->downloadInvoice($invoiceId, [
        'vendor' => 'Your Company',
        'product' => 'Your Product',
        'street' => 'Main Str. 1',
        'location' => '2000 Antwerp, Belgium',
        'phone' => '+32 499 00 00 00',
        'email' => 'info@example.com',
        'url' => 'https://example.com',
        'vendorVat' => 'BE123456789',
    ]);

Phương thức `downloadInvoice` cũng cho phép đặt tên file tùy chỉnh thông qua tham số thứ ba của nó. Tên file này sẽ tự động có hậu tố là `.pdf`:

    return $request->user()->downloadInvoice($invoiceId, [], 'my-invoice');

<a name="custom-invoice-render"></a>
#### Custom Invoice Renderer

Cashier cũng cho phép bạn sử dụng tùy chỉnh trình tạo hóa đơn. Mặc định, Cashier sử dụng implementation `DompdfInvoiceRenderer`, sử dụng thư viện PHP [dompdf](https://github.com/dompdf/dompdf) để tạo hóa đơn cho Cashier. Tuy nhiên, bạn có thể sử dụng bất kỳ trình tạo nào mà bạn muốn bằng cách implementation interface `Laravel\Cashier\Contracts\InvoiceRenderer`. Ví dụ: bạn có thể muốn hiển thị PDF hóa đơn bằng cách sử dụng lệnh gọi API tới service hiển thị PDF của third-party:

    use Illuminate\Support\Facades\Http;
    use Laravel\Cashier\Contracts\InvoiceRenderer;
    use Laravel\Cashier\Invoice;

    class ApiInvoiceRenderer implements InvoiceRenderer
    {
        /**
         * Render the given invoice and return the raw PDF bytes.
         *
         * @param  \Laravel\Cashier\Invoice. $invoice
         * @param  array  $data
         * @param  array  $options
         * @return string
         */
        public function render(Invoice $invoice, array $data = [], array $options = []): string
        {
            $html = $invoice->view($data)->render();

            return Http::get('https://example.com/html-to-pdf', ['html' => $html])->get()->body();
        }
    }

Khi bạn đã implement xong contract tạo hóa đơn, bạn nên cập nhật giá trị cấu hình `cashier.invoices.renderer` trong file cấu hình `config/cashier.php` của ứng dụng. Giá trị cấu hình này phải được set thành tên class implement trình tạo hoá đơn tùy chỉnh của bạn.

<a name="checkout"></a>
## Checkout

Cashier Stripe cũng hỗ trợ cho [Stripe Checkout](https://stripe.com/payments/checkout). Stripe Checkout sẽ giảm bớt những khó khăn khi implement các trang tùy chỉnh để chấp nhận thanh toán bằng cách cung cấp các trang thanh toán có sẵn.

Tài liệu sau đây sẽ chứa các thông tin về cách bắt đầu sử dụng Stripe Checkout với Cashier. Để tìm hiểu thêm về Stripe Checkout, bạn cũng nên xem qua [tài liệu riêng của Stripe về Checkout](https://stripe.com/docs/payments/checkout).

<a name="product-checkouts"></a>
### Product Checkouts

Bạn có thể thực hiện thanh toán cho một sản phẩm hiện được tạo trong bảng điều khiển Stripe của bạn bằng phương thức `checkout` trên một billable model. Phương thức `checkout` sẽ bắt đầu một session Stripe Checkout mới. Mặc định, bạn bắt buộc phải truyền một ID giá Stripe:

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout('price_tshirt');
    });

Nếu cần, bạn cũng có thể chỉ định số lượng sản phẩm:

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 15]);
    });

Khi một khách hàng truy cập vào route này, họ sẽ được chuyển hướng đến trang thanh toán của Stripe. Mặc định, khi người dùng hoàn thành hoặc hủy mua hàng, họ sẽ được chuyển hướng đến vị trí route `home` của bạn, nhưng bạn có thể chỉ định các URL được gọi lại này bằng cách sử dụng các tùy chọn `success_url` và `cancel_url`:

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 1], [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
    });

Khi định nghĩa tùy chọn thanh toán `success_url`, bạn có thể hướng dẫn Stripe thêm một ID session thanh toán làm một tham số trong URL khi gọi URL này của bạn. Để làm như vậy, hãy thêm chuỗi ký tự `{CHECKOUT_SESSION_ID}` vào key `success_url` của bạn. Stripe sẽ thay thế biến này bằng ID session thanh toán thực tế:

    use Illuminate\Http\Request;
    use Stripe\Checkout\Session;
    use Stripe\Customer;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 1], [
            'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
            'cancel_url' => route('checkout-cancel'),
        ]);
    });

    Route::get('/checkout-success', function (Request $request) {
        $checkoutSession = $request->user()->stripe()->checkout->sessions->retrieve($request->get('session_id'));

        return view('checkout.success', ['checkoutSession' => $checkoutSession]);
    })->name('checkout-success');

<a name="checkout-promotion-codes"></a>
#### Promotion Codes

Mặc định, Stripe Checkout không cho phép [mã khuyến mại cho người dùng](https://stripe.com/docs/billing/subscriptions/discounts/codes). May mắn thay, có một cách dễ dàng để bật những tính năng này cho trang thanh toán của bạn. Để làm như vậy, bạn có thể gọi phương thức `allowPromotionCodes`:

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()
            ->allowPromotionCodes()
            ->checkout('price_tshirt');
    });

<a name="single-charge-checkouts"></a>
### Single Charge Checkouts

Bạn cũng có thể thực hiện một khoản phí đơn giản cho một sản phẩm đặc biệt chưa được tạo trong bảng điều khiển Stripe của bạn. Để làm như vậy, bạn có thể sử dụng phương thức `checkoutCharge` trên một billable model và truyền cho nó số tiền, tên sản phẩm và số lượng tùy chọn. Khi khách hàng truy cập vào route này, họ sẽ được chuyển hướng đến trang thanh toán của Stripe:

    use Illuminate\Http\Request;

    Route::get('/charge-checkout', function (Request $request) {
        return $request->user()->checkoutCharge(1200, 'T-Shirt', 5);
    });

> **Warning**
> Khi sử dụng phương thức `checkoutCharge`, Stripe sẽ luôn tạo ra một sản phẩm và một giá mới trong trang tổng quan Stripe của bạn. Do đó, chúng tôi khuyên bạn nên tạo trước các sản phẩm trong bảng điều khiển Stripe của bạn và sử dụng phương thức `checkout` để thay thế.

<a name="subscription-checkouts"></a>
### Subscription Checkouts

> **Warning**
> Sử dụng Stripe Checkout cho subscription yêu cầu bạn phải bật webhook `customer.subscription.created` trong trang tổng quan Stripe của bạn. Webhook này sẽ tạo ra một record subscription trong cơ sở dữ liệu của bạn và lưu trữ tất cả các mục subscription có liên quan.

Bạn cũng có thể sử dụng Stripe Checkout để bắt đầu một subscription. Sau khi định nghĩa subscription của bạn bằng các phương thức tạo subscription của Cashier, bạn có thể gọi phương thức `checkout `. Khi khách hàng truy cập route này, họ sẽ được chuyển hướng đến trang thanh toán của Stripe:

    use Illuminate\Http\Request;

    Route::get('/subscription-checkout', function (Request $request) {
        return $request->user()
            ->newSubscription('default', 'price_monthly')
            ->checkout();
    });

Cũng giống như với thanh toán sản phẩm, bạn có thể tùy chỉnh các URL success và cancel:

    use Illuminate\Http\Request;

    Route::get('/subscription-checkout', function (Request $request) {
        return $request->user()
            ->newSubscription('default', 'price_monthly')
            ->checkout([
                'success_url' => route('your-success-route'),
                'cancel_url' => route('your-cancel-route'),
            ]);
    });

Tất nhiên, bạn cũng có thể kích hoạt mã khuyến mãi để thanh toán subscription:

    use Illuminate\Http\Request;

    Route::get('/subscription-checkout', function (Request $request) {
        return $request->user()
            ->newSubscription('default', 'price_monthly')
            ->allowPromotionCodes()
            ->checkout();
    });

> **Warning**
> Rất tiếc, Stripe Checkout không hỗ trợ tất cả các tùy chọn thanh toán subscription khi subscription được bắt đầu. Việc sử dụng phương thức `anchorBillingCycleOn` trên subscription builder, sẽ set hành vi tính phí cho khoảng thời gian còn lại hoặc là sẽ set hành vi thanh toán sẽ không bị ảnh hưởng trong các session Stripe Checkout. Vui lòng tham khảo [tài liệu về API session của Stripe Checkout](https://stripe.com/docs/api/checkout/sessions/create) để xem qua những thông số nào sẽ khả dụng.

<a name="stripe-checkout-trial-periods"></a>
#### Stripe Checkout & Trial Periods

Tất nhiên, bạn có thể định nghĩa một thời gian dùng thử khi xây dựng một subscription bằng Stripe Checkout:

    $checkout = Auth::user()->newSubscription('default', 'price_monthly')
        ->trialDays(3)
        ->checkout();

Tuy nhiên, thời gian dùng thử ít nhất phải là 48 giờ, đây là lượng thời gian dùng thử tối thiểu mà Stripe Checkout hỗ trợ.

<a name="stripe-checkout-subscriptions-and-webhooks"></a>
#### Subscriptions & Webhooks

Hãy nhớ rằng Stripe và Cashier cập nhật trạng thái subscription qua webhook, vì vậy có khả năng subscription sẽ chưa hoạt động khi khách hàng quay lại ứng dụng sau khi nhập thông tin thanh toán của họ. Để xử lý tình huống này, bạn có thể muốn hiển thị một thông báo thông báo cho người dùng rằng thanh toán hoặc subscription của họ đang chờ xử lý.

<a name="collecting-tax-ids"></a>
### Collecting Tax IDs

Checkout cũng hỗ trợ thu thập Tax ID của khách hàng. Để bật tính năng này trong một session thanh toán, hãy gọi phương thức `collectTaxIds` khi tạo session:

    $checkout = $user->collectTaxIds()->checkout('price_tshirt');

Khi phương thức này được gọi, một checkbox sẽ được hiển thị cho khách hàng cho phép họ biết liệu họ có đang mua hàng với tư cách là một công ty hay không. Nếu có, họ sẽ cung cấp Tax ID của họ.

> **Warning**
> Nếu bạn đã cấu hình [thu thuế tự động](#tax-configuration) trong service provider của ứng dụng thì tính năng này sẽ được bật tự động và không cần gọi phương thức `collectTaxIds`.

<a name="guest-checkouts"></a>
### Guest Checkouts

Sử dụng phương thức `Checkout::guest`, bạn có thể bắt đầu các session thanh toán cho những khách hàng không có "tài khoản" trong ứng dụng của bạn:

    use Illuminate\Http\Request;
    use Laravel\Cashier\Checkout;

    Route::get('/product-checkout', function (Request $request) {
        return Checkout::guest()->create('price_tshirt', [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
    });

Tương tự như khi tạo session thanh toán cho người dùng hiện tại, bạn có thể sử dụng thêm các phương thức có sẵn trên instance session `Laravel\Cashier\CheckoutBuilder` để tùy chỉnh session thanh toán cho khách hàng:

    use Illuminate\Http\Request;
    use Laravel\Cashier\Checkout;

    Route::get('/product-checkout', function (Request $request) {
        return Checkout::guest()
            ->withPromotionCode('promo-code')
            ->create('price_tshirt', [
                'success_url' => route('your-success-route'),
                'cancel_url' => route('your-cancel-route'),
            ]);
    });

Sau khi quá trình thanh toán hoàn tất, Stripe có thể gửi sự kiện webhook `checkout.session.completed`, vì vậy hãy đảm bảo là bạn đã [cấu hình webhook Stripe](https://dashboard.stripe.com/webhooks) để thực sự gửi sự kiện này vào ứng dụng của bạn. Sau khi bật webhook trong bảng điều khiển Stripe, bạn có thể [xử lý webhook bằng Cashier](#handling-stripe-webhooks). Đối tượng có trong payload của webhook này sẽ là một [đối tượng `checkout`](https://stripe.com/docs/api/checkout/sessions/object) mà bạn có thể dùng để kiểm tra đơn đặt hàng của khách hàng.

<a name="handling-failed-payments"></a>
## Xử lý lỗi thanh toán

Thỉnh thoảng, thanh toán cho các subscription hoặc các khoản phí một lần có thể thất bại. Khi điều này xảy ra, Cashier sẽ đưa ra một ngoại lệ là `Laravel\Cashier\Exceptions\IncompletePayment` để thông báo cho bạn biết rằng ngoại lệ đã xảy ra. Sau khi xử lý được ngoại lệ này, bạn có hai tùy chọn về cách tiếp tục.

Một là, bạn có thể chuyển hướng khách hàng của bạn đến trang xác nhận thanh toán chuyên dụng đi kèm với Cashier. Trang này đã liên kết với một named route được đăng ký thông qua service provider của Cashier. Vì vậy, bạn có thể catch ngoại lệ `IncompletePayment` và chuyển hướng người dùng đến trang xác nhận thanh toán:

    use Laravel\Cashier\Exceptions\IncompletePayment;

    try {
        $subscription = $user->newSubscription('default', 'price_monthly')
                                ->create($paymentMethod);
    } catch (IncompletePayment $exception) {
        return redirect()->route(
            'cashier.payment',
            [$exception->payment->id, 'redirect' => route('home')]
        );
    }

Trên trang xác nhận thanh toán, khách hàng sẽ được nhắc nhập lại thông tin thẻ tín dụng của họ và thực hiện thêm bất kỳ hành động nào theo yêu cầu của Stripe, chẳng hạn như xác nhận "3D Secure". Sau khi xác nhận thanh toán, người dùng sẽ được chuyển hướng đến URL được cung cấp bởi thông số `redirect` được chỉ định ở trên. Khi chuyển hướng, các biến url `message` (string) và `success` (integer) sẽ được thêm vào URL. Trang thanh toán hiện tại hỗ trợ các loại phương thức thanh toán sau:

<div class="content-list" markdown="1">

- Credit Cards
- Alipay
- Bancontact
- BECS Direct Debit
- EPS
- Giropay
- iDEAL
- SEPA Direct Debit

</div>

Ngoài ra, bạn có thể cho phép Stripe xử lý xác nhận thanh toán cho bạn. Trong trường hợp này, thay vì chuyển hướng đến trang xác nhận thanh toán, bạn có thể [thiết lập email thanh toán tự động của Stripe](https://dashboard.stripe.com/account/billing/automatic) trong trang tổng quan Stripe của bạn. Tuy nhiên, nếu ngoại lệ `IncompletePayment` bị đưa ra, bạn vẫn nên thông báo cho người dùng rằng họ sẽ nhận được email có thêm hướng dẫn xác nhận thanh toán.

Các ngoại lệ thanh toán có thể được đưa ra cho các phương thức sau: `charge`, `invoiceFor` và `invoice` trên các model sử dụng trait `Billable`. Khi tương tác với subscription, phương thức `create` trên `SubscriptionBuilder` và phương thức `incrementAndInvoice` và `swapAndInvoice` trên các model `Subscription` và `SubscriptionItem` có thể tạo ra các ngoại lệ thanh toán chưa hoàn thành.

Việc xác định xem subscription hiện tại có khoản thanh toán chưa hoàn thành hay không có thể được thực hiện bằng cách sử dụng phương thức `hasIncompletePayment` trên một billable model hoặc một instance subscription:

    if ($user->hasIncompletePayment('default')) {
        //
    }

    if ($user->subscription('default')->hasIncompletePayment()) {
        //
    }

Bạn có thể lấy được trạng thái cụ thể của khoản thanh toán chưa hoàn tất bằng cách kiểm tra thuộc tính `payment` trong instance ngoại lệ:

    use Laravel\Cashier\Exceptions\IncompletePayment;

    try {
        $user->charge(1000, 'pm_card_threeDSecure2Required');
    } catch (IncompletePayment $exception) {
        // Get the payment intent status...
        $exception->payment->status;

        // Check specific conditions...
        if ($exception->payment->requiresPaymentMethod()) {
            // ...
        } elseif ($exception->payment->requiresConfirmation()) {
            // ...
        }
    }

<a name="strong-customer-authentication"></a>
## Strong Customer Authentication

Nếu doanh nghiệp của bạn hoặc khách hàng của bạn có trụ sở tại Châu Âu, bạn sẽ cần tuân thủ các quy định về Strong Customer Authentication (SCA) của EU. Các quy định này đã được Liên minh Châu Âu áp dụng vào tháng 9 năm 2019 để ngăn chặn gian lận thanh toán. May mắn thay, Stripe và Cashier đã chuẩn bị sẵn sàng để xây dựng các ứng dụng tuân thủ SCA.

> **Warning**
> Trước khi bắt đầu, hãy xem [hướng dẫn của Stripe về PSD2 và SCA](https://stripe.com/guides/strong-customer-authentication) cũng như [tài liệu về API SCA mới](https://stripe.com/docs/strong-customer-authentication) của họ.

<a name="payments-requiring-additional-confirmation"></a>
### Payments Requiring Additional Confirmation

Các quy định của SCA thường yêu cầu xác minh thêm để xác nhận và xử lý một khoản thanh toán. Khi điều này xảy ra, Cashier sẽ đưa ra một ngoại lệ `Laravel\Cashier\Exceptions\IncompletePayment` để thông báo cho bạn biết rằng cần phải xác minh thêm. Để tìm thấy thêm thông tin về cách xử lý cho các trường hợp ngoại lệ này bạn có thể tìm thấy trong tài liệu về [xử lý các khoản thanh toán không thành công](#handling-failed-payments).

Payment confirmation screens presented by Stripe or Cashier may be tailored to a specific bank or card issuer's payment flow and can include additional card confirmation, a temporary small charge, separate device authentication, or other forms of verification.

<a name="incomplete-and-past-due-state"></a>
#### Incomplete and Past Due State

Khi một khoản thanh toán yêu cầu xác nhận bổ sung, subscription sẽ vẫn ở trạng thái `incomplete` hoặc `past_due`, là giá trị cột `stripe_status`. Cashier sẽ tự động kích hoạt subscription của khách hàng ngay sau khi xác nhận thanh toán hoàn tất và ứng dụng của bạn sẽ được Stripe thông báo qua webhook về việc hoàn thành.

Để biết thêm thông tin về trạng thái `incomplete` và `past_due`, vui lòng tham khảo [thêm tài liệu của chúng tôi về các trạng thái](#incomplete-and-past-due-status).

<a name="off-session-payment-notifications"></a>
### Thông báo thanh toán Off-Session

Vì các quy định của SCA yêu cầu khách hàng thỉnh thoảng cần xác minh chi tiết thanh toán của họ ngay cả khi subscription của họ đang hoạt động, nên Cashier có thể gửi thông báo cho khách hàng khi yêu cầu xác nhận thanh toán off-session. Ví dụ: điều này có thể xảy ra khi subscription đang được gia hạn. Thông báo thanh toán của Cashier có thể được bật bằng cách set biến môi trường `CASHIER_PAYMENT_NOTIFICATION` thành một class thông báo. Mặc định, thông báo này sẽ bị tắt. Tất nhiên, Cashier có chứa một class thông báo mà bạn có thể sử dụng cho mục đích này, nhưng bạn có thể tự do thêm class thông báo của bạn nếu muốn:

```ini
CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment
```

Để đảm bảo rằng thông báo xác nhận thanh toán off-session sẽ được gửi, hãy chắc chắn rằng [Stripe webhooks đã được cấu hình](#handling-stripe-webhooks) cho ứng dụng của bạn và webhook `invoice.payment_action_required` được bật trong trang tổng quan Stripe của bạn. Ngoài ra, model `Billable` của bạn cũng nên sử dụng trait `Illuminate\Notifications\Notifiable` của Laravel.

> **Warning**
> Thông báo sẽ được gửi ngay cả khi khách hàng đang tự thực hiện thanh toán và nhận yêu cầu xác nhận bổ sung. Thật không may, không có cách nào để Stripe biết rằng một khoản thanh toán là được một khách hàng tự thực hiện hay là thông qua "off-session". Tuy nhiên, khách hàng sẽ chỉ nhận được thông báo "Thanh toán thành công" nếu họ truy cập vào trang thanh toán sau khi đã xác nhận thanh toán của mình. Khách hàng sẽ không được phép xác nhận cùng một khoản thanh toán tới hai lần và chịu khoản phí đến lần thứ hai.

<a name="stripe-sdk"></a>
## Stripe SDK

Nhiều đối tượng của Cashier là các wrapper của các đối tượng Stripe SDK. Nếu bạn muốn tương tác trực tiếp với các đối tượng Stripe, bạn có thể lấy ra chúng bằng phương thức `asStripe`:

    $stripeSubscription = $subscription->asStripeSubscription();

    $stripeSubscription->application_fee_percent = 5;

    $stripeSubscription->save();

Bạn cũng có thể sử dụng phương thức `updateStripeSubscription` để cập nhật trực tiếp một subscription Stripe:

    $subscription->updateStripeSubscription(['application_fee_percent' => 5]);

Bạn có thể gọi phương thức `stripe` trên class `Cashier` nếu bạn muốn sử dụng trực tiếp client `Stripe\StripeClient`. Ví dụ: bạn có thể sử dụng phương thức này để truy cập vào instance `StripeClient` và lấy ra danh sách giá từ tài khoản Stripe của bạn:

    use Laravel\Cashier\Cashier;

    $prices = Cashier::stripe()->prices->all();

<a name="testing"></a>
## Testing

Khi testing một ứng dụng sử dụng Cashier, bạn có thể cần mô phỏng các request HTTP thực tế đối với API của Stripe; tuy nhiên, điều này đòi hỏi bạn phải thực hiện lại một phần hành vi của chính Cashier. Do đó, chúng tôi khuyên bạn nên cho phép các bài test của bạn được chạm vào các API Stripe thực tế. Mặc dù điều này sẽ chậm hơn, nhưng nó sẽ cung cấp thêm niềm tin rằng ứng dụng của bạn đang hoạt động như mong đợi và bất kỳ bài test chậm nào cũng có thể được lưu vào trong một group testing PHPUnit của riêng nó.

Khi testing, hãy nhớ rằng bản thân Cashier đã có sẵn một bộ test case tuyệt vời, vì vậy bạn chỉ nên tập trung vào việc test các luồng subscription và thanh toán của ứng dụng của riêng bạn chứ không phải test mọi hành vi cơ bản của Cashier.

Để bắt đầu, hãy thêm phiên bản **testing** của Stripe secret vào file `phpunit.xml` của bạn:

    <env name="STRIPE_SECRET" value="sk_test_<your-key>"/>

Bây giờ, bất cứ khi nào bạn tương tác với Cashier trong khi testing, nó sẽ gửi các request API thực tế đến môi trường testing của Stripe của bạn. Để thuận tiện, bạn nên tạo ra trước các subscription và các price cho tài khoản testing Stripe của bạn mà sau đó bạn có thể sử dụng các subscription đó hoặc các price đó trong quá trình testing.

> **Note**
> Để test nhiều tình huống thanh toán khác nhau, chẳng hạn như thẻ tín dụng bị từ chối và thất bại, bạn có thể sử dụng [số thẻ và token dành cho test](https://stripe.com/docs/testing) được cung cấp bởi Stripe.
