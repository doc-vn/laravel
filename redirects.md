# HTTP Redirects

- [Tạo Redirects](#creating-redirects)
- [Redirect tới tên route](#redirecting-named-routes)
- [Redirect tới Controller Action](#redirecting-controller-actions)
- [Redirect với Flashed Session Data](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## Tạo Redirects

Redirect response là một instance của class `Illuminate\Http\RedirectResponse` và chứa các header thích hợp cần thiết để chuyển hướng người dùng đến một URL khác. Có một số cách để tạo ra một instance `RedirectResponse`. Phương thức đơn giản nhất là sử dụng helper global `redirect`:

    Route::get('/dashboard', function () {
        return redirect('/home/dashboard');
    });

Thỉnh thoảng bạn có thể muốn chuyển hướng người dùng đến vị trí trước đó của họ, chẳng hạn như khi form đã gửi không hợp lệ. Bạn có thể làm như vậy bằng cách sử dụng hàm helper global `back`. Vì tính năng này sử dụng [session](/docs/{{version}}/session), nên hãy đảm bảo rằng route mà được gọi trong hàm `back` cũng đang sử dụng group middleware `web` hoặc áp dụng tất cả middleware session:

    Route::post('/user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
## Redirect tới tên route

Khi bạn gọi helper `redirect` không có tham số, một instance của `Illuminate\Routing\Redirector` sẽ được trả về, cho phép bạn gọi bất kỳ phương thức nào trong instance `Redirector`. Ví dụ: để tạo một `RedirectResponse` cho tên một route, bạn có thể sử dụng phương thức `route`:

    return redirect()->route('login');

Nếu route của bạn có các tham số, thì bạn có thể truyền chúng làm tham số thứ hai cho phương thức `route`:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

Để thuận tiện, Laravel cũng cung cấp một hàm global function `to_route`:

    return to_route('profile', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>
#### Populating Parameters Via Eloquent Models

Nếu bạn đang chuyển hướng đến một route có tham số "ID" đang được khai báo từ một model Eloquent, bạn có thể truyền qua chính model đó. ID sẽ được trích xuất tự động:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

Nếu bạn muốn tùy biến giá trị được set trong tham số route, bạn nên ghi đè phương thức `getRouteKey` trong model Eloquent của bạn:

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
## Redirect tới Controller Action

Bạn cũng có thể tạo chuyển hướng đến [controller actions](/docs/{{version}}/controllers). Để làm điều đó, bạn hãy truyền vào một controller và tên action của nó cho phương thức `action`.

    use App\Http\Controllers\HomeController;

    return redirect()->action([HomeController::class, 'index']);

Nếu controller route của bạn yêu cầu tham số, bạn có thể truyền chúng làm tham số thứ hai cho phương thức `action`:

    return redirect()->action(
        [UserController::class, 'profile'], ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
## Redirect với Flashed Session Data

Chuyển hướng đến một URL mới và [flashing data tới session](/docs/{{version}}/session#flash-data) thường được thực hiện cùng một lúc. Thông thường, điều này được thực hiện sau khi thực hiện thành công một hành động nào đó thì bạn sẽ cần flash một message success đến session. Để thuận tiện, bạn có thể tạo một instance `RedirectResponse` và flash data tới session chỉ trong một chuỗi phương thức duy nhất:

    Route::post('/user/profile', function () {
        // Update the user's profile...

        return redirect('/dashboard')->with('status', 'Profile updated!');
    });

Bạn có thể sử dụng phương thức `withInput` được cung cấp bởi instance `RedirectResponse` để flash dữ liệu input của request hiện tại vào session trước khi chuyển hướng người dùng đến một vị trí mới. Sau khi dữ liệu input đã được chuyển sang session xong, bạn có thể dễ dàng [lấy nó ra](/docs/{{version}}/requests#retrieving-old-input) trong lần request tiếp theo:

    return back()->withInput();

Sau khi người dùng đã được chuyển hướng, bạn có thể hiển thị message đã được flash từ [session](/docs/{{version}}/session). Ví dụ: sử dụng [cú pháp Blade](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif
