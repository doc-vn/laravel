# Database: Seeding

- [Giới thiệu](#introduction)
- [Viết Seeder](#writing-seeders)
    - [Dùng Model Factory](#using-model-factories)
    - [Gọi thêm file Seeder](#calling-additional-seeders)
    - [Tắt Model Event](#muting-model-events)
- [Chạy Seeder](#running-seeders)

<a name="introduction"></a>
## Giới thiệu

Laravel có khả năng tạo cơ sở dữ liệu cùng dữ liệu bằng các class seed. Tất cả các class seed được lưu trong thư mục `database/seeders`. Mặc định, một class `DatabaseSeeder` đã được định nghĩa sẵn cho bạn. Từ class này, bạn có thể sử dụng phương thức `call` để chạy các class seed khác nhau của bạn, cho phép bạn kiểm soát thứ tự được tạo.

> **Note**
> [Chế độ bảo vệ mass assignment](/docs/{{version}}/eloquent#mass-assignment) sẽ tự động bị vô hiệu hóa trong quá trình tạo cơ sở dữ liệu.

<a name="writing-seeders"></a>
## Viết Seeder

Để tạo một file seed, hãy chạy [Lệnh Artisan](/docs/{{version}}/artisan) `make:seeder`. Tất cả các seeder được tạo ra bởi lệnh này sẽ được lưu trong thư mục `database/seeders`:

```shell
php artisan make:seeder UserSeeder
```

Một class seeder chỉ chứa một phương thức mặc định là: `run`. Phương thức này được gọi khi [Lệnh Artisan](/docs/{{version}}/artisan) `db:seed` được chạy. Trong phương thức `run`, bạn có thể thêm dữ liệu vào cơ sở dữ liệu của bạn theo cách mà bạn mong muốn. Bạn có thể sử dụng [query builder](/docs/{{version}}/queries) để thêm dữ liệu theo cách thủ công hoặc bạn có thể sử dụng [Eloquent model factories](/docs/{{version}}/eloquent-factories).

Ví dụ, hãy sửa class `DatabaseSeeder` mặc định và thêm một câu số câu lệnh thêm cơ sở dữ liệu vào trong phương thức `run`:

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeders.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => Str::random(10),
                'email' => Str::random(10).'@gmail.com',
                'password' => Hash::make('password'),
            ]);
        }
    }

> **Note**
> Bạn có thể khai báo bất kỳ phụ thuộc nào mà bạn cần trong phương thức `run`. Những phụ thuộc đó sẽ được tự động resolve thông qua Laravel [service container](/docs/{{version}}/container).

<a name="using-model-factories"></a>
### Dùng Model Factory

Tất nhiên, việc khai báo thủ công các thuộc tính cho từng model seed sẽ rất cồng kềnh. Thay vào đó, bạn có thể sử dụng [model factories](/docs/{{version}}/eloquent-factories) để tạo ra một lượng lớn các bản ghi cho cơ sở dữ liệu. Trước tiên, hãy xem lại [tài liệu model factory](/docs/{{version}}/eloquent-factories) để tìm hiểu cách định nghĩa factory cho bạn. Khi bạn đã định nghĩa các factory của bạn rồi, bạn có thể sử dụng hàm helper `factory` để thêm các bản ghi vào trong cơ sở dữ liệu.

Ví dụ: hãy tạo 50 người dùng và mỗi người dùng đó có một quan hệ post:

    use App\Models\User;

    /**
     * Run the database seeders.
     *
     * @return void
     */
    public function run()
    {
        User::factory()
                ->count(50)
                ->hasPosts(1)
                ->create();
    }

<a name="calling-additional-seeders"></a>
### Gọi thêm file Seeder

Trong class `DatabaseSeeder`, bạn có thể sử dụng phương thức `call` để gọi các class seed khác. Sử dụng phương thức `call` cho phép bạn chia nhỏ cơ sở dữ liệu của bạn thành nhiều file, để không có một class seeder nào trở nên quá lớn. Phương thức `call` chấp nhận một mảng các class seeder sẽ được thực thi:

    /**
     * Run the database seeders.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
            CommentSeeder::class,
        ]);
    }

<a name="muting-model-events"></a>
### Tắt Model Event

Trong khi chạy seed, bạn có thể muốn ngăn các model gửi event. Bạn có thể thực hiện điều này bằng cách sử dụng trait `WithoutModelEvents`. Khi được sử dụng, trait `WithoutModelEvents` này sẽ đảm bảo không có event model nào được gửi đi, ngay cả khi các class seed bổ sung được thực thi thông qua chạy phương thức `call`:

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Console\Seeds\WithoutModelEvents;

    class DatabaseSeeder extends Seeder
    {
        use WithoutModelEvents;

        /**
         * Run the database seeders.
         *
         * @return void
         */
        public function run()
        {
            $this->call([
                UserSeeder::class,
            ]);
        }
    }

<a name="running-seeders"></a>
## Chạy Seeder

Bạn có thể chạy lệnh Artisan `db:seed` để tạo cơ sở dữ liệu cho bạn. Mặc định, lệnh `db:seed` sẽ chạy trên class `Database\Seeders\DatabaseSeeder` để có thể gọi sang các class seed khác. Tuy nhiên, bạn có thể sử dụng tùy chọn `--class` để chỉ định một class seeder cụ thể sẽ được chạy:

```shell
php artisan db:seed

php artisan db:seed --class=UserSeeder
```

Bạn cũng có thể tạo lại cơ sở dữ liệu cho bạn bằng lệnh `migrate:fresh` cùng với tùy chọn `--seed`, lệnh này sẽ xoá tất cả các bảng và chạy lại tất cả các migration của bạn. Nó sẽ hữu ích khi bạn building lại hoàn toàn cơ sở dữ liệu của bạn. Tùy chọn `--seeder` có thể được sử dụng để chỉ định một seeder cụ thể sẽ được chạy:

```shell
php artisan migrate:fresh --seed

php artisan migrate:fresh --seed --seeder=UserSeeder
```

<a name="forcing-seeding-production"></a>
#### Buộc Seeder phải chạy trong môi trường production

Một số thao tác seeding có thể khiến dữ liệu của bạn bị thay đổi hoặc bị mất. Để bảo vệ bạn khỏi việc chạy các lệnh seeding trên các cơ sở dữ liệu production, bạn sẽ được nhắc xác nhận trước khi seeder được chạy trong môi trường production. Để buộc seeder chạy mà không có nhắc xác nhận, hãy sử dụng flag `--force`:

```shell
php artisan db:seed --force
```
