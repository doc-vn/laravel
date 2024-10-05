# Rate Limiting

- [Giới thiệu](#introduction)
    - [Cấu hình Cache](#cache-configuration)
- [Cách sử dụng cơ bản](#basic-usage)
    - [Tự tăng số lần thử](#manually-incrementing-attempts)
    - [Xoá số lần thử](#clearing-attempts)

<a name="introduction"></a>
## Giới thiệu

Laravel có chứa một abstraction giới hạn tỷ lệ dễ sử dụng, kết hợp với [cache](cache) của ứng dụng của bạn, cung cấp một cách dễ dàng để giới hạn bất kỳ hành động nào trong một khoảng thời gian nhất định.

> [!NOTE]
> Nếu bạn quan tâm đến việc giới hạn tỷ lệ các request HTTP, vui lòng tham khảo [tài liệu về middleware giới hạn tỷ lệ](routing#rate-limiting).

<a name="cache-configuration"></a>
### Cấu hình Cache

Thông thường, bộ giới hạn tỷ lệ sẽ sử dụng bộ nhớ cache mặc định của bạn như bạn đã được định nghĩa bởi khóa `default` trong file cấu hình `cache` của ứng dụng của bạn. Tuy nhiên, bạn có thể chỉ định driver cache nào mà bộ giới hạn tỷ lệ sẽ sử dụng bằng cách định nghĩa khóa `limiter` trong file cấu hình `cache` của ứng dụng của bạn:

    'default' => 'memcached',

    'limiter' => 'redis',

<a name="basic-usage"></a>
## Cách sử dụng cơ bản

Facade `Illuminate\Support\Facades\RateLimiter` có thể được sử dụng để tương tác với bộ giới hạn tỷ lệ. Phương thức đơn giản nhất được cung cấp bởi bộ giới hạn tỷ lệ là phương thức `attempt`, phương thức này giới hạn tỷ lệ một callback nhất định trong một số giây nhất định.

Phương thức `attempt` sẽ trả về `false` khi lệnh callback không còn lần thử nào nữa; nếu không, phương thức `attempt` vẫn sẽ trả về kết quả của lệnh callback hoặc `true`. Tham số đầu tiên được phương thức `attempt` chấp nhận là một "khóa" giới hạn tốc độ, có thể là bất kỳ chuỗi nào bạn chọn để thể hiện hành động bị giới hạn tốc độ:

    use Illuminate\Support\Facades\RateLimiter;

    $executed = RateLimiter::attempt(
        'send-message:'.$user->id,
        $perMinute = 5,
        function() {
            // Send message...
        }
    );

    if (! $executed) {
      return 'Too many messages sent!';
    }

Nếu cần, bạn có thể cung cấp thêm tham số thứ tư cho phương thức `attempt`, đó là "thời gian reset", hoặc là số giây mà các lần thử sẽ được reset lại khi chúng thử xong. Ví dụ, chúng ta có thể sửa ví dụ trên để cho phép chạy năm lần thử sau mỗi hai phút:

    $executed = RateLimiter::attempt(
        'send-message:'.$user->id,
        $perTwoMinutes = 5,
        function() {
            // Send message...
        },
        $decayRate = 120,
    );

<a name="manually-incrementing-attempts"></a>
### Tự tăng số lần thử

Nếu bạn muốn tương tác thủ công với bộ giới hạn tỷ lệ, thì có nhiều phương thức khác nhau. Ví dụ: bạn có thể gọi phương thức `tooManyAttempts` để xác định xem khóa giới hạn tỷ lệ nhất định có vượt quá số lần thử tối đa được phép mỗi phút hay không:

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
        return 'Too many attempts!';
    }

    RateLimiter::increment('send-message:'.$user->id);

    // Send message...

Ngoài ra, bạn có thể sử dụng phương thức `remaining` để lấy ra số lần thử còn lại cho một khóa nhất định. Nếu một khóa nhất định còn số lần thử lại, bạn có thể gọi phương thức `increment` để tăng tổng số lần thử:

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
        RateLimiter::increment('send-message:'.$user->id);

        // Send message...
    }

If you would like to increment the value for a given rate limiter key by more than one, you may provide the desired amount to the `increment` method:

    RateLimiter::increment('send-message:'.$user->id, amount: 5);

<a name="determining-limiter-availability"></a>
#### Determining Limiter Availability

Khi một khóa không còn lần thử nào nữa, phương thức `availableIn` sẽ trả về số giây còn lại cho đến khi có nhiều lần thử hơn:

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
        $seconds = RateLimiter::availableIn('send-message:'.$user->id);

        return 'You may try again in '.$seconds.' seconds.';
    }

    RateLimiter::increment('send-message:'.$user->id);

    // Send message...

<a name="clearing-attempts"></a>
### Xoá số lần thử

Bạn có thể set lại số lần thử cho một khóa giới hạn tỷ lệ nhất định bằng phương thức `clear`. Ví dụ: bạn có thể set lại số lần thử khi người nhận đã đọc một tin nhắn nhất định:

    use App\Models\Message;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Mark the message as read.
     */
    public function read(Message $message): Message
    {
        $message->markAsRead();

        RateLimiter::clear('send-message:'.$message->user_id);

        return $message;
    }
