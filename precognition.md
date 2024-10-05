# Precognition

- [Giới thiệu](#introduction)
- [Validation trực tiếp](#live-validation)
    - [Dùng với Vue](#using-vue)
    - [Dùng với Vue và Inertia](#using-vue-and-inertia)
    - [Dùng với React](#using-react)
    - [Dùng với React và Inertia](#using-react-and-inertia)
    - [Dùng với Alpine và Blade](#using-alpine)
    - [Cấu hình Axios](#configuring-axios)
- [Tuỳ chỉnh Validation Rules](#customizing-validation-rules)
- [Xử lý File Uploads](#handling-file-uploads)
- [Quản lý Side-Effects](#managing-side-effects)
- [Testing](#testing)

<a name="introduction"></a>
## Giới thiệu

Laravel Precognition cho phép bạn dự đoán kết quả của một request HTTP khi gửi lên server. Một trong những ứng dụng chính của Precognition là khả năng cung cấp xác thực "trực tiếp" cho ứng dụng JavaScript frontend mà không cần phải copy các quy tắc validation từ backend của ứng dụng. Precognition kết hợp đặc biệt tốt với [starter kits](/docs/{{version}}/starter-kits) dựa trên Inertia của Laravel.

Khi Laravel nhận được một "request precognitive", nó sẽ thực hiện tất cả các middleware của route và resolve các library controller của route, bao gồm cả validation [request form](/docs/{{version}}/validation#form-request-validation) - nhưng nó sẽ không thực sự thực thi bất kỳ phương thức nào của controller.

<a name="live-validation"></a>
## Validation trực tiếp

<a name="using-vue"></a>
### Dùng với Vue

Sử dụng Laravel Precognition, bạn có thể cung cấp trải nghiệm xác thực trực tiếp cho người dùng mà không cần phải sao chép các quy tắc validation sang ứng dụng Vue frontend. Để minh họa cách thức hoạt động, hãy cùng xây dựng một form tạo người dùng mới trong ứng dụng của chúng ta.

Đầu tiên, để bật Precognition cho một route, middleware `HandlePrecognitiveRequests` cần được thêm vào định nghĩa route. Bạn cũng nên tạo một [form request](/docs/{{version}}/validation#form-request-validation) để chứa các quy tắc validation của route:

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Tiếp theo, bạn nên cài đặt helper frontend Laravel Precognition cho Vue thông qua NPM:

```shell
npm install laravel-precognition-vue
```

Sau khi cài đặt package Laravel Precognition xong, bây giờ bạn có thể tạo một đối tượng form bằng hàm `useForm` của Precognition, cung cấp phương thức HTTP (`post`) với URL target (`/users`) và dữ liệu form ban đầu.

Sau đó, để kích hoạt xác thực trực tiếp, hãy gọi phương thức `validate` của form qua event `change` của input và cung cấp thêm tên của input:

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit();
</script>

<template>
    <form @submit.prevent="submit">
        <label for="name">Name</label>
        <input
            id="name"
            v-model="form.name"
            @change="form.validate('name')"
        />
        <div v-if="form.invalid('name')">
            {{ form.errors.name }}
        </div>

        <label for="email">Email</label>
        <input
            id="email"
            type="email"
            v-model="form.email"
            @change="form.validate('email')"
        />
        <div v-if="form.invalid('email')">
            {{ form.errors.email }}
        </div>

        <button :disabled="form.processing">
            Create User
        </button>
    </form>
</template>
```

Bây giờ, khi người dùng điền vào form, Precognition sẽ cung cấp output xác thực trực tiếp dựa trên các quy tắc validation request form của route. Khi dữ liệu input của form thay đổi, nó sẽ chờ cho đến khi người dùng nhập xong, nếu người dùng đã nhập xong, thì một request xác thực "precognition" sẽ được gửi đến ứng dụng Laravel của bạn. Bạn có thể cấu hình thời gian chờ để gửi bằng cách gọi hàm `setValidationTimeout` của form:

```js
form.setValidationTimeout(3000);
```

Khi request xác thực đang được xử lý, thuộc tính `validating` của property sẽ là `true`:

```html
<div v-if="form.validating">
    Validating...
</div>
```

Bất kỳ lỗi xác thực nào được trả về trong quá trình request xác thực hoặc trong quá trình gửi form, đều sẽ được tự động điền vào đối tượng `errors` của form:

```html
<div v-if="form.invalid('email')">
    {{ form.errors.email }}
</div>
```

Bạn có thể xác định xem form có lỗi nào không bằng cách sử dụng thuộc tính `hasErrors` của form:

```html
<div v-if="form.hasErrors">
    <!-- ... -->
</div>
```

Bạn cũng có thể xác định xem dữ liệu input đã được xác thực thành công hay không bằng cách truyền tên của dữ liệu input đó cho các hàm `valid` và `invalid` của form:

```html
<span v-if="form.valid('email')">
    ✅
</span>

<span v-else-if="form.invalid('email')">
    ❌
</span>
```

> [!WARNING]
> Thông tin input của form chỉ được coi là hợp lệ hoặc không hợp lệ sau khi đã thay đổi và nhận được một response validation.

Nếu bạn đang validate một tập hợp dữ liệu input form thông qua Precognition, việc xóa lỗi có thể hữu ích. Bạn có thể sử dụng hàm `forgetError` của form để thực hiện việc này:

```html
<input
    id="avatar"
    type="file"
    @change="(e) => {
        form.avatar = e.target.files[0]

        form.forgetError('avatar')
    }"
>
```

Tất nhiên, bạn cũng có thể chạy code để tương tác lại với response cho việc gửi form. Hàm `submit` của form trả về một Axios request promise. Điều này sẽ cung cấp một cách thuận tiện để bạn có thể truy cập vào payload của response, reset lại các input của form khi gửi thành công hoặc xử lý khi không thành công:

```js
const submit = () => form.submit()
    .then(response => {
        form.reset();

        alert('User created.');
    })
    .catch(error => {
        alert('An error occurred.');
    });
```

Bạn có thể xác định một request form có đang được xử lý hay không bằng cách kiểm tra thuộc tính `processing` của form:

```html
<button :disabled="form.processing">
    Submit
</button>
```

<a name="using-vue-and-inertia"></a>
### Dùng với Vue và Inertia

> [!NOTE]
> Nếu bạn muốn bắt đầu phát triển ứng dụng Laravel với Vue và Inertia, hãy cân nhắc sử dụng một trong các [bộ khởi động starter kit](/docs/{{version}}/starter-kits) của chúng tôi. Bộ khởi động starter kit của Laravel sẽ cung cấp một nền tảng xác thực gồm backend và frontend cho ứng dụng Laravel của bạn.

Trước khi sử dụng Precognition với Vue và Inertia, hãy nhớ xem qua tài liệu chung của chúng tôi về [sử dụng Precognition với Vue](#using-vue). Khi sử dụng Vue với Inertia, bạn sẽ cần cài đặt thư viện Precognition tương thích với Inertia thông qua NPM:

```shell
npm install laravel-precognition-vue-inertia
```

Sau khi cài đặt xong, hàm `useForm` của Precognition sẽ trả về một [helper form](https://inertiajs.com/forms#form-helper) Inertia với các chức năng xác thực đã được nói ở trên.

Phương thức `submit` của helper form đã được tinh giản, loại bỏ các nhu cầu về chỉ định phương thức HTTP hoặc URL. Thay vào đó, bạn có thể truyền một [tùy chọn truy cập](https://inertiajs.com/manual-visits) của Inertia làm tham số đầu tiên và duy nhất. Ngoài ra, phương thức `submit` sẽ không trả về Promise như trong ví dụ Vue ở trên. Mà thay vào đó, bạn có thể cung cấp bất kỳ [event callback](https://inertiajs.com/manual-visits#event-callbacks) nào được Inertia hỗ trợ vào trong các tùy chọn truy cập được cung cấp cho phương thức `submit`:

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue-inertia';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit({
    preserveScroll: true,
    onSuccess: () => form.reset(),
});
</script>
```

<a name="using-react"></a>
### Dùng với React

Sử dụng Laravel Precognition, bạn có thể cung cấp trải nghiệm xác thực trực tiếp cho người dùng mà không cần phải sao chép các quy tắc validation sang ứng dụng React frontend. Để minh họa cách thức hoạt động, hãy cùng xây dựng một form tạo người dùng mới trong ứng dụng của chúng ta.

Đầu tiên, để bật Precognition cho một route, middleware `HandlePrecognitiveRequests` cần được thêm vào định nghĩa route. Bạn cũng nên tạo một [form request](/docs/{{version}}/validation#form-request-validation) để chứa các quy tắc validation của route:

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Tiếp theo, bạn nên cài đặt helper frontend Laravel Precognition cho React thông qua NPM:

```shell
npm install laravel-precognition-react
```

Sau khi cài đặt package Laravel Precognition xong, bây giờ bạn có thể tạo một đối tượng form bằng hàm `useForm` của Precognition, cung cấp phương thức HTTP (`post`) với URL target (`/users`) và dữ liệu form ban đầu.

Để enable xác thực trực tiếp, bạn nên listen event `change` và event `blur` của từng input. Trong handler event `change`, bạn nên thiết lập dữ liệu của form bằng hàm `setData`, truyền vào tên input và giá trị mới. Sau đó, trong handler event `blur`, hãy gọi phương thức `validate` của form và cung cấp tên input:

```jsx
import { useForm } from 'laravel-precognition-react';

export default function Form() {
    const form = useForm('post', '/users', {
        name: '',
        email: '',
    });

    const submit = (e) => {
        e.preventDefault();

        form.submit();
    };

    return (
        <form onSubmit={submit}>
            <label for="name">Name</label>
            <input
                id="name"
                value={form.data.name}
                onChange={(e) => form.setData('name', e.target.value)}
                onBlur={() => form.validate('name')}
            />
            {form.invalid('name') && <div>{form.errors.name}</div>}

            <label for="email">Email</label>
            <input
                id="email"
                value={form.data.email}
                onChange={(e) => form.setData('email', e.target.value)}
                onBlur={() => form.validate('email')}
            />
            {form.invalid('email') && <div>{form.errors.email}</div>}

            <button disabled={form.processing}>
                Create User
            </button>
        </form>
    );
};
```

Bây giờ, khi người dùng điền vào form, Precognition sẽ cung cấp output xác thực trực tiếp dựa trên các quy tắc validation request form của route. Khi dữ liệu input của form thay đổi, nó sẽ chờ cho đến khi người dùng nhập xong, nếu người dùng đã nhập xong, thì một request xác thực "precognition" sẽ được gửi đến ứng dụng Laravel của bạn. Bạn có thể cấu hình thời gian chờ để gửi bằng cách gọi hàm `setValidationTimeout` của form:

```js
form.setValidationTimeout(3000);
```

Khi request xác thực đang được xử lý, thuộc tính `validating` của property sẽ là `true`:

```jsx
{form.validating && <div>Validating...</div>}
```

Bất kỳ lỗi xác thực nào được trả về trong quá trình request xác thực hoặc trong quá trình gửi form, đều sẽ được tự động điền vào đối tượng `errors` của form:

```jsx
{form.invalid('email') && <div>{form.errors.email}</div>}
```

Bạn có thể xác định xem form có lỗi nào không bằng cách sử dụng thuộc tính `hasErrors` của form:

```jsx
{form.hasErrors && <div><!-- ... --></div>}
```

Bạn cũng có thể xác định xem dữ liệu input đã được xác thực thành công hay không bằng cách truyền tên của dữ liệu input đó cho các hàm `valid` và `invalid` của form:

```jsx
{form.valid('email') && <span>✅</span>}

{form.invalid('email') && <span>❌</span>}
```

> [!WARNING]
> Thông tin input của form chỉ được coi là hợp lệ hoặc không hợp lệ sau khi đã thay đổi và nhận được một response validation.

Nếu bạn đang validate một tập hợp dữ liệu input form thông qua Precognition, việc xóa lỗi có thể hữu ích. Bạn có thể sử dụng hàm `forgetError` của form để thực hiện việc này:

```jsx
<input
    id="avatar"
    type="file"
    onChange={(e) =>
        form.setData('avatar', e.target.value);

        form.forgetError('avatar');
    }
>
```

Tất nhiên, bạn cũng có thể chạy code để tương tác lại với response cho việc gửi form. Hàm `submit` của form trả về một Axios request promise. Điều này sẽ cung cấp một cách thuận tiện để bạn có thể truy cập vào payload của response, reset lại các input của form khi gửi thành công hoặc xử lý khi không thành công:

```js
const submit = (e) => {
    e.preventDefault();

    form.submit()
        .then(response => {
            form.reset();

            alert('User created.');
        })
        .catch(error => {
            alert('An error occurred.');
        });
};
```

Bạn có thể xác định một request form có đang được xử lý hay không bằng cách kiểm tra thuộc tính `processing` của form:

```html
<button disabled={form.processing}>
    Submit
</button>
```

<a name="using-react-and-inertia"></a>
### Dùng với React và Inertia

> [!NOTE]
> Nếu bạn muốn bắt đầu phát triển ứng dụng Laravel với React và Inertia, hãy cân nhắc sử dụng một trong các [bộ khởi động starter kit](/docs/{{version}}/starter-kits) của chúng tôi. Bộ khởi động starter kit của Laravel sẽ cung cấp một nền tảng xác thực gồm backend và frontend cho ứng dụng Laravel của bạn.

Trước khi sử dụng Precognition với React và Inertia, hãy nhớ xem qua tài liệu chung của chúng tôi về [sử dụng Precognition với React](#using-react). Khi sử dụng React với Inertia, bạn sẽ cần cài đặt thư viện Precognition tương thích với Inertia thông qua NPM:

```shell
npm install laravel-precognition-react-inertia
```

Sau khi cài đặt xong, hàm `useForm` của Precognition sẽ trả về một [helper form](https://inertiajs.com/forms#form-helper) Inertia với các chức năng xác thực đã được nói ở trên.

Phương thức `submit` của helper form đã được tinh giản, loại bỏ các nhu cầu về chỉ định phương thức HTTP hoặc URL. Thay vào đó, bạn có thể truyền một [tùy chọn truy cập](https://inertiajs.com/manual-visits) của Inertia làm tham số đầu tiên và duy nhất. Ngoài ra, phương thức `submit` sẽ không trả về Promise như trong ví dụ React ở trên. Mà thay vào đó, bạn có thể cung cấp bất kỳ [event callback](https://inertiajs.com/manual-visits#event-callbacks) nào được Inertia hỗ trợ vào trong các tùy chọn truy cập được cung cấp cho phương thức `submit`:

```js
import { useForm } from 'laravel-precognition-react-inertia';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = (e) => {
    e.preventDefault();

    form.submit({
        preserveScroll: true,
        onSuccess: () => form.reset(),
    });
};
```

<a name="using-alpine"></a>
### Dùng với Alpine và Blade

Sử dụng Laravel Precognition, bạn có thể cung cấp trải nghiệm xác thực trực tiếp cho người dùng mà không cần phải sao chép các quy tắc validation sang ứng dụng Alpine frontend. Để minh họa cách thức hoạt động, hãy cùng xây dựng một form tạo người dùng mới trong ứng dụng của chúng ta.

Đầu tiên, để bật Precognition cho một route, middleware `HandlePrecognitiveRequests` cần được thêm vào định nghĩa route. Bạn cũng nên tạo một [form request](/docs/{{version}}/validation#form-request-validation) để chứa các quy tắc validation của route:

```php
use App\Http\Requests\CreateUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (CreateUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Tiếp theo, bạn nên cài đặt helper frontend Laravel Precognition cho Alpine thông qua NPM:

```shell
npm install laravel-precognition-alpine
```

Sau đó, hãy đăng ký plugin Precognition với Alpine trong file `resources/js/app.js` của bạn:

```js
import Alpine from 'alpinejs';
import Precognition from 'laravel-precognition-alpine';

window.Alpine = Alpine;

Alpine.plugin(Precognition);
Alpine.start();
```

Sau khi cài đặt và đăng ký thành công package Laravel Precognition, bây giờ bạn có thể tạo đối tượng form bằng phương thức "magic" `$form` của Precognition, cung cấp phương thức HTTP (`post`), URL đích (`/users`) và dữ liệu form ban đầu.

Để enable xác thực trực tiếp, bạn nên liên kết dữ liệu của form với input tương ứng và sau đó listen event `change` của từng input. Trong handler event `change`, bạn nên gọi phương thức `validate` của form và cung cấp tên input:

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '',
        email: '',
    }),
}">
    @csrf
    <label for="name">Name</label>
    <input
        id="name"
        name="name"
        x-model="form.name"
        @change="form.validate('name')"
    />
    <template x-if="form.invalid('name')">
        <div x-text="form.errors.name"></div>
    </template>

    <label for="email">Email</label>
    <input
        id="email"
        name="email"
        x-model="form.email"
        @change="form.validate('email')"
    />
    <template x-if="form.invalid('email')">
        <div x-text="form.errors.email"></div>
    </template>

    <button :disabled="form.processing">
        Create User
    </button>
</form>
```

Bây giờ, khi người dùng điền vào form, Precognition sẽ cung cấp output xác thực trực tiếp dựa trên các quy tắc validation request form của route. Khi dữ liệu input của form thay đổi, nó sẽ chờ cho đến khi người dùng nhập xong, nếu người dùng đã nhập xong, thì một request xác thực "precognition" sẽ được gửi đến ứng dụng Laravel của bạn. Bạn có thể cấu hình thời gian chờ để gửi bằng cách gọi hàm `setValidationTimeout` của form:

```js
form.setValidationTimeout(3000);
```

Khi request xác thực đang được xử lý, thuộc tính `validating` của property sẽ là `true`:

```html
<template x-if="form.validating">
    <div>Validating...</div>
</template>
```

Bất kỳ lỗi xác thực nào được trả về trong quá trình request xác thực hoặc trong quá trình gửi form, đều sẽ được tự động điền vào đối tượng `errors` của form:

```html
<template x-if="form.invalid('email')">
    <div x-text="form.errors.email"></div>
</template>
```

Bạn có thể xác định xem form có lỗi nào không bằng cách sử dụng thuộc tính `hasErrors` của form:

```html
<template x-if="form.hasErrors">
    <div><!-- ... --></div>
</template>
```

Bạn cũng có thể xác định xem dữ liệu input đã được xác thực thành công hay không bằng cách truyền tên của dữ liệu input đó cho các hàm `valid` và `invalid` của form:

```html
<template x-if="form.valid('email')">
    <span>✅</span>
</template>

<template x-if="form.invalid('email')">
    <span>❌</span>
</template>
```

> [!WARNING]
> Thông tin input của form chỉ được coi là hợp lệ hoặc không hợp lệ sau khi đã thay đổi và nhận được một response validation.

Bạn có thể xác định một request form có đang được xử lý hay không bằng cách kiểm tra thuộc tính `processing` của form:

```html
<button :disabled="form.processing">
    Submit
</button>
```

<a name="repopulating-old-form-data"></a>
#### Repopulating Old Form Data

Trong ví dụ tạo người dùng đã thảo luận ở trên, chúng ta sử dụng Precognition để thực hiện xác thực trực tiếp; tuy nhiên, chúng ta đang thực hiện gửi form về phía server theo cách truyền thống. Vì vậy, form cần được điền đầy đủ thông tin input và lỗi validation "old" cũng được trả về từ quá trình gửi form lên phía server:

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '{{ old('name') }}',
        email: '{{ old('email') }}',
    }).setErrors({{ Js::from($errors->messages()) }}),
}">
```

Ngoài ra, nếu bạn muốn gửi form qua XHR, bạn có thể sử dụng hàm `submit` của form, hàm này trả về một Axios request promise:

```html
<form
    x-data="{
        form: $form('post', '/register', {
            name: '',
            email: '',
        }),
        submit() {
            this.form.submit()
                .then(response => {
                    form.reset();

                    alert('User created.')
                })
                .catch(error => {
                    alert('An error occurred.');
                });
        },
    }"
    @submit.prevent="submit"
>
```

<a name="configuring-axios"></a>
### Cấu hình Axios

Thư viện xác thực Precognition sử dụng HTTP client [Axios](https://github.com/axios/axios) để gửi request đến backend của ứng dụng. Để thuận tiện, bạn có thể tùy chỉnh instance Axios nếu ứng dụng của bạn yêu cầu. Ví dụ: khi sử dụng thư viện `laravel-precognition-vue`, bạn có thể thêm request header vào mỗi request gửi đi trong file `resources/js/app.js` của ứng dụng:

```js
import { client } from 'laravel-precognition-vue';

client.axios().defaults.headers.common['Authorization'] = authToken;
```

Hoặc, nếu bạn đã cấu hình instance Axios cho ứng dụng của bạn, bạn có thể yêu cầu Precognition sử dụng instance đó:

```js
import Axios from 'axios';
import { client } from 'laravel-precognition-vue';

window.axios = Axios.create()
window.axios.defaults.headers.common['Authorization'] = authToken;

client.use(window.axios)
```

> [!WARNING]
> Các thư viện Precognition mà theo cấu trúc Inertia sẽ chỉ sử dụng instance Axios đã được cấu hình cho các request xác thực. Còn việc gửi form sẽ luôn được gửi bởi Inertia.

<a name="customizing-validation-rules"></a>
## Tuỳ chỉnh Validation Rules

Bạn có thể tùy chỉnh các quy tắc xác thực được thực hiện trong một precognitive request bằng cách sử dụng phương thức `isPrecognitive` của request.

Ví dụ, trên một form tạo người dùng, chúng ta có thể chỉ muốn validate một mật khẩu "dễ bị lộ" khi lần gửi form cuối cùng. Đối với các precognitive validation request, chúng ta chỉ cần xác thực rằng mật khẩu là bắt buộc và có tối thiểu 8 ký tự. Sử dụng phương thức `isPrecognitive`, chúng ta có thể tùy chỉnh các quy tắc được định nghĩa bởi form request của bạn:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class StoreUserRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    protected function rules()
    {
        return [
            'password' => [
                'required',
                $this->isPrecognitive()
                    ? Password::min(8)
                    : Password::min(8)->uncompromised(),
            ],
            // ...
        ];
    }
}
```

<a name="handling-file-uploads"></a>
## Xử lý File Uploads

Mặc định, Laravel Precognition không upload hoặc validate file trong một precognitive validation request. Điều này đảm bảo các file lớn sẽ không bị upload nhiều lần một cách không cần thiết.

Bởi vậy, bạn nên đảm bảo là ứng dụng của bạn đã được [tùy biến các quy tắc xác thực của form request](#customizing-validation-rules) để yêu cầu field được chỉ định chỉ khi form được gửi lên thật sự:

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
protected function rules()
{
    return [
        'avatar' => [
            ...$this->isPrecognitive() ? [] : ['required'],
            'image',
            'mimes:jpg,png'
            'dimensions:ratio=3/2',
        ],
        // ...
    ];
}
```

Nếu bạn muốn thêm các file vào trong mọi request xác thực, bạn có thể gọi hàm `validateFiles` trên instance form của bạn:

```js
form.validateFiles();
```

<a name="managing-side-effects"></a>
## Quản lý Side-Effects

Khi thêm middleware `HandlePrecognitiveRequests` vào một route, bạn nên cân nhắc xem có bất kỳ tác dụng phụ nào khác có trong một middleware _khác_ mà bạn cần phải bỏ qua trong khi một precognitive request được gửi hay không.

Ví dụ, bạn có thể có một middleware tăng số lượng "tương tác" mà mỗi người dùng có thể thực hiện với ứng dụng của bạn, nhưng bạn lại không muốn các precognitive request được tính là một tương tác. Để thực hiện điều này, chúng ta có thể kiểm tra phương thức `isPrecognitive` của request trước khi tăng số lượng tương tác:

```php
<?php

namespace App\Http\Middleware;

use App\Facades\Interaction;
use Closure;
use Illuminate\Http\Request;

class InteractionMiddleware
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): mixed
    {
        if (! $request->isPrecognitive()) {
            Interaction::incrementFor($request->user());
        }

        return $next($request);
    }
}
```

<a name="testing"></a>
## Testing

Nếu bạn muốn thực hiện các precognitive request trong các bài kiểm tra của bạn, Laravel `TestCase` có chứa sẵn một helper `withPrecognition` sẽ thêm header `Precognition` vào trong request.

Ngoài ra, nếu bạn muốn kiểm tra một precognitive request là thành công và không trả về bất kỳ lỗi xác thực nào, bạn có thể sử dụng phương thức `assertSuccessfulPrecognition` trên response:

```php
public function test_it_validates_registration_form_with_precognition()
{
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();
    $this->assertSame(0, User::count());
}
```
