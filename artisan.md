# Artisan Console

- [Giới thiệu](#introduction)
    - [Tinker (REPL)](#tinker)
- [Viết Command](#writing-commands)
    - [Tạo Command](#generating-commands)
    - [Cấu trúc Command](#command-structure)
    - [Closure Command](#closure-commands)
- [Định nghĩa Input](#defining-input-expectations)
    - [Tham số](#arguments)
    - [Tuỳ chọn](#options)
    - [Input cho một mảng](#input-arrays)
    - [Thêm mô tả cho Input](#input-descriptions)
- [Input và output của Command](#command-io)
    - [Lấy giá trị input](#retrieving-input)
    - [Hỏi giá trị input](#prompting-for-input)
    - [Viết Output](#writing-output)
- [Đăng ký Command](#registering-commands)
- [Chạy command bên ngoài CLI](#programmatically-executing-commands)
    - [Gọi Command từ một Command khác](#calling-commands-from-other-commands)
- [Xử lý tín hiệu](#signal-handling)
- [Tùy chỉnh Stub](#stub-customization)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Artisan là một giao diện dòng lệnh đi kèm với Laravel. Artisan tồn tại ở gốc của ứng dụng của bạn dưới dạng một tập lệnh `artisan` và cung cấp một số lệnh hữu ích có thể hỗ trợ bạn trong khi bạn xây dựng application. Để xem danh sách tất cả các lệnh Artisan có sẵn, bạn có thể sử dụng lệnh `list`:

    php artisan list

Mỗi lệnh cũng chứa một lệnh "help" để hiển thị và mô tả các tùy chọn và các tham số dành cho lệnh đó. Để xem lệnh help, hãy set `help` vào trước tên của command:

    php artisan help migrate

<a name="laravel-sail"></a>
#### Laravel Sail

Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail) làm môi trường phát triển local của bạn, hãy nhớ sử dụng dòng lệnh `sail` để gọi các lệnh Artisan. Sail sẽ thực hiện các lệnh Artisan của bạn trong các Docker container của ứng dụng của bạn:

    ./sail artisan list

<a name="tinker"></a>
### Tinker (REPL)

Laravel Tinker là một REPL mạnh mẽ cho Laravel framework, cung cấp bởi package [PsySH](https://github.com/bobthecow/psysh).

<a name="installation"></a>
#### Installation

Mặc định tất cả các ứng dụng Laravel đều chứa Tinker. Tuy nhiên, bạn có thể cài đặt Tinker thông qua Composer nếu trước đó bạn đã xóa nó ra khỏi ứng dụng của bạn:

    composer require laravel/tinker

> {tip} Nếu bạn đang tìm một tool giao diện người dùng để tương tác với ứng dụng Laravel của bạn? Hãy xem [Tinkerwell](https://tinkerwell.app)!

<a name="usage"></a>
#### Usage

Tinker cho phép bạn tương tác trực tiếp với toàn bộ application Laravel của bạn trên command line, bao gồm cả model Eloquent, job, event, vv... Để vào được môi trường Tinker, hãy chạy lệnh Artisan `tinker`:

    php artisan tinker

Bạn có thể export file cấu hình của Tinker bằng lệnh `vendor:publish`:

    php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"

> {note} Hàm helper `dispatch` và phương thức `dispatch` trên class `Dispatchable` phụ thuộc vào việc thu gom rác để set job vào queue. Do đó, khi sử dụng tinker, bạn nên sử dụng `Bus::dispatch` hoặc `Queue::push` để điều phối job.

<a name="command-allow-list"></a>
#### Command Allow List

Tinker có sử dụng một danh sách "allow" để xác định các lệnh Artisan nào được phép chạy. Mặc định, bạn có thể chạy các lệnh `clear-compiled`, `down`, `env`, `inspire`, `migrate`, `optimize`, và `up`. Nếu bạn muốn cho phép thêm các lệnh khác, bạn có thể thêm chúng vào mảng `commands` trong file cấu hình `tinker.php` của bạn:

    'commands' => [
        // App\Console\Commands\ExampleCommand::class,
    ],

<a name="classes-that-should-not-be-aliased"></a>
#### Classes That Should Not Be Aliased

Thông thường, Tinker sẽ tự động đặt bí danh cho các class khi bạn tương tác với chúng trong Tinker. Tuy nhiên, bạn có thể muốn không đặt bí danh cho một số class. Bạn có thể thực hiện điều này bằng cách thêm các class đó vào trong mảng `dont_alias` của file cấu hình `tinker.php` của bạn:

    'dont_alias' => [
        App\Models\User::class,
    ],

<a name="writing-commands"></a>
## Viết Commands

Ngoài các lệnh được cung cấp với Artisan, bạn có thể tự xây dựng các lệnh của riêng bạn. Các lệnh thường được lưu trữ trong thư mục `app/Console/Commands`; tuy nhiên, bạn cũng có thể thoải mái chọn vị trí lưu trữ mà bạn muốn, miễn là các lệnh của bạn có thể load được bởi Composer.

<a name="generating-commands"></a>
### Tạo Commands

Để tạo một lệnh mới, bạn có thể sử dụng lệnh Artisan `make:command`. Lệnh này sẽ tạo một class command mới trong thư mục `app/Console/Commands`. Đừng lo lắng nếu thư mục này không tồn tại trong application của bạn, vì nó sẽ được tạo vào lần đầu tiên bạn chạy lệnh Artisan `make:command`:

    php artisan make:command SendEmails

<a name="command-structure"></a>
### Cấu trúc Command

Sau khi đã tạo xong command, bạn hãy định nghĩa các giá trị phù hợp cho các thuộc tính `signature` và `description`. Các thuộc tính này sẽ được hiển thị thông tin command của bạn trên màn hình `list`. Thuộc tính `signature` cũng cho phép bạn định nghĩa [kỳ vọng input đầu vào cho command của bạn](#defining-input-expectations). Phương thức `handle` sẽ được gọi khi lệnh của bạn được thực thi. Bạn có thể cài đặt logic của bạn vào trong phương thức này.

Chúng ta hãy xem một ví dụ về command. Lưu ý rằng chúng ta có thể yêu cầu bất kỳ service nào mà chúng ta muốn thông qua hàm `handle` của command. Laravel [service container](/docs/{{version}}/container) sẽ tự động inject tất cả các phụ thuộc đã được khai báo có trong phương thức đó:

    <?php

    namespace App\Console\Commands;

    use App\Models\User;
    use App\Support\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'mail:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send a marketing email to a user';

        /**
         * Create a new command instance.
         *
         * @return void
         */
        public function __construct()
        {
            parent::__construct();
        }

        /**
         * Execute the console command.
         *
         * @param  \App\Support\DripEmailer  $drip
         * @return mixed
         */
        public function handle(DripEmailer $drip)
        {
            $drip->send(User::find($this->argument('user')));
        }
    }

> {tip} Để code của bạn có thể tái sử dụng tốt hơn, thì cách tốt nhất là giữ cho các command của bạn được "nhẹ" và hãy để các application service hoàn thành nhiệm vụ đó cho bạn. Trong ví dụ dưới trên, hãy chú ý rằng chúng ta sẽ inject một service class để thực hiện một "công việc nặng" như việc gửi e-mail.

<a name="closure-commands"></a>
### Closure Command

Các command được tạo dựa trên closure sẽ cung cấp thêm một giải pháp để định nghĩa các command. Giống như cách mà các closure route làm, là tạo thêm một cách định nghĩa cho controller, bạn hãy nghĩ các closure command này như là một cách định nghĩa khác cho các class command, thay vì phải tạo ra một file command mới. Trong phương thức `commands` ở trong file `app/Console/Kernel.php` của bạn, Laravel sẽ load sẵn file `routes/console.php`:

    /**
     * Register the closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

Mặc dù file này không định nghĩa các HTTP route, nhưng nó định nghĩa các closure dựa theo format của route vào trong application của bạn. Trong file này, bạn có thể định nghĩa tất cả các closure dựa trên lệnh console của bạn bằng phương thức `Artisan::command`. Phương thức `command` chấp nhận hai tham số: một là một [command signature](#defining-input-expectations) và hai là một closure để nhận vào các tham số và các option của command:

    Artisan::command('mail:send {user}', function ($user) {
        $this->info("Sending email to: {$user}!");
    });

Closure sẽ được liên kết với một instance command cơ bản, nên bạn có toàn quyền truy cập vào tất cả các phương thức helper mà bạn thường dùng trên một class command cơ bản.

<a name="type-hinting-dependencies"></a>
#### Khai báo dạng kiểu phụ thuộc

Ngoài việc nhận vào các tham số và các option của command, closure command cũng có thể khai báo thêm các phụ thuộc mà bạn muốn resolve từ [service container](/docs/{{version}}/container):

    use App\Models\User;
    use App\Support\DripEmailer;

    Artisan::command('mail:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

<a name="closure-command-descriptions"></a>
#### Closure Command Descriptions

Khi định nghĩa một command dựa trên closure, bạn có thể sử dụng phương thức `purpose` để thêm mô tả cho command. Mô tả này sẽ được hiển thị khi bạn chạy lệnh `php artisan list` hoặc lệnh `php artisan help`:

    Artisan::command('mail:send {user}', function ($user) {
        // ...
    })->purpose('Send a marketing email to a user');

<a name="defining-input-expectations"></a>
## Định nghĩa Input

Khi viết một lệnh console, thường thu nhận các dữ liệu đầu vào từ người dùng thông qua các tham số hoặc các option. Laravel làm cho nó rất thuận tiện để xác định đầu vào mà bạn mong muốn từ người dùng bằng cách sử dụng thuộc tính `signature` trên mỗi lệnh của bạn. Thuộc tính `signature` cho phép bạn định nghĩa tên, tham số và các option cho lệnh theo một cú pháp đơn giản, dễ hiểu, giống như cú pháp trên route.

<a name="arguments"></a>
### Tham số

Tất cả các tham số và các tùy chọn do người dùng cung cấp được wrap trong một dấu ngoặc nhọn. Trong ví dụ sau, lệnh sẽ định nghĩa một tham số bắt buộc: `user`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

Bạn cũng có thể tạo ra tham số tùy chọn hoặc định nghĩa giá trị mặc định cho các tham số đó:

    // Optional argument...
    mail:send {user?}

    // Optional argument with default value...
    mail:send {user=foo}

<a name="options"></a>
### Tuỳ chọn

Tùy chọn, giống như một tham số, là một dạng khác của input user. Các tùy chọn sẽ được gán tiền tố với hai dấu gạch nối (`--`) khi chúng được cung cấp thông qua cửa sổ dòng lệnh. Có hai loại tùy chọn: loại tùy chọn nhận một giá trị và loại tuỳ chọn không nhận giá trị nào. Các tùy chọn không nhận giá trị đóng vai trò như là một "switch" boolean. Chúng ta hãy xem một ví dụ về loại tùy chọn này:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send {user} {--queue}';

Trong ví dụ này, switch `--queue` có thể được chỉ định khi gọi lệnh Artisan. Nếu switch `--queue` được thông qua, giá trị của tùy chọn sẽ là `true`. Nếu không, giá trị sẽ là `false`:

    php artisan mail:send 1 --queue

<a name="options-with-values"></a>
#### Tuỳ chọn với giá trị

Tiếp theo, chúng ta hãy xem một tùy chọn nhận một giá trị. Nếu người dùng phải chỉ định một giá trị cho một tùy chọn, thì bạn hãy thêm hậu tố vào tên của tùy chọn đó bằng dấu `=`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send {user} {--queue=}';

Trong ví dụ này, người dùng có thể truyền một giá trị cho tùy chọn đó như sau. Nếu tùy chọn không được truyền vào khi chạy command, thì giá trị của nó sẽ là `null`:

    php artisan mail:send 1 --queue=default

Bạn cũng có thể gán một giá trị mặc định cho các tùy chọn này bằng cách chỉ định giá trị mặc định sau tên mỗi tùy chọn. Nếu không có giá trị tùy chọn nào được người dùng truyền vào, thì giá trị mặc định sẽ được sử dụng:

    mail:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### Option Shortcuts

Để gán một shortcut khi định nghĩa một tùy chọn, bạn có thể chỉ định nó vào phía trước tên của một tùy chọn và sử dụng ký tự `|` như một dấu để phân tách shortcut khỏi toàn bộ tên tùy chọn:

    mail:send {user} {--Q|queue}

Khi gọi command trên terminal của bạn, các shortcut tùy chọn phải được set bằng một dấu gạch ngang ở đằng trước:

    php artisan mail:send 1 -Q

<a name="input-arrays"></a>
### Input cho một mảng

Nếu bạn muốn định nghĩa các tham số hoặc tùy chọn để nhận vào nhiều giá trị, bạn có thể sử dụng ký tự `*`. Đầu tiên, chúng ta hãy xem một ví dụ định nghĩa một tham số như sau:

    mail:send {user*}

Khi gọi phương thức này, các tham số `user` có thể được truyền theo dòng lệnh. Ví dụ: lệnh sau sẽ set giá trị của `user` thành một mảng với `foo` và `bar` là các giá trị của nó:

    php artisan mail:send foo bar

Ký tự `*` này có thể được kết hợp với một định nghĩa tùy chọn tham số để cho phép nhập từ không đến nhiều instance tham số:

    mail:send {user?*}

<a name="option-arrays"></a>
#### Option Arrays

Khi định nghĩa một tùy chọn yêu cầu nhiều giá trị input, mỗi giá trị tùy chọn đó được truyền đến command phải được đặt tên tùy chọn đó ở đằng trước:

    mail:send {user} {--id=*}

    php artisan mail:send --id=1 --id=2

<a name="input-descriptions"></a>
### Thêm mô tả cho Input

Bạn có thể gán một mô tả cho các input đầu vào như tham số hoặc tùy chọn bằng cách tách tên tham số đó ra khỏi mô tả bằng dấu hai chấm. Nếu bạn cần thêm một chút chỗ trống để định nghĩa thêm cho lệnh của mình, hãy định nghĩa nó trên nhiều dòng:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send
                            {user : The ID of the user}
                            {--queue : Whether the job should be queued}';

<a name="command-io"></a>
## Input và output của Command

<a name="retrieving-input"></a>
### Lấy giá trị input

Trong khi lệnh của bạn đang thực thi, bạn có thể sẽ cần truy cập vào các giá trị của các tham số và các tùy chọn đã được khai báo trong lệnh của bạn. Để làm như vậy, bạn có thể sử dụng các phương thức `argument` và `option`. Nếu một tham số hoặc tùy chọn không tồn tại, thì giá trị `null` sẽ được trả về:

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

Nếu bạn cần lấy ra tất cả các tham số dưới dạng một `array`, hãy gọi phương thức `arguments`:

    $arguments = $this->arguments();

Các tùy chọn có thể được lấy ra dễ dàng như các tham số bằng cách sử dụng phương thức `option`. Để lấy tất cả các tùy chọn dưới dạng một mảng, hãy gọi phương thức `options`:

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // // Retrieve all options as an array...
    $options = $this->options();

<a name="prompting-for-input"></a>
### Hỏi giá trị input

Ngoài việc hiển thị output, bạn cũng có thể yêu cầu người dùng cung cấp thêm thông tin trong quá trình đang thực thi lệnh. Phương thức `ask` sẽ hỏi người dùng với một câu hỏi có sẵn, chấp nhận thông tin nhập thêm của người dùng và sau đó truyền lại thông tin mới nhập thêm đó cho lệnh của bạn:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

Phương thức `secret` tương tự như phương thức `ask`, nhưng đầu vào của người dùng sẽ không được hiển thị cho họ khi họ gõ vào console. Phương thức này hữu ích khi yêu cầu thông tin nhạy cảm như mật khẩu:

    $password = $this->secret('What is the password?');

<a name="asking-for-confirmation"></a>
#### Xác nhận

Nếu bạn cần yêu cầu người dùng xác nhận "yes hoặc no", bạn có thể sử dụng phương thức `confirm`. Mặc định, phương thức này sẽ trả về `false`. Tuy nhiên, nếu người dùng nhập `y` hoặc `yes` để trả lời confirm, thì phương thức sẽ trả về `true`.

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

Nếu cần thiết, bạn có thể chỉ định rằng mặc định confirm sẽ trả về giá trị `true` bằng cách truyền giá trị `true` làm tham số thứ hai cho phương thức `confirm`:

    if ($this->confirm('Do you wish to continue?', true)) {
        //
    }

<a name="auto-completion"></a>
#### Auto-Completion

Phương thức `anticipate` có thể được sử dụng để cung cấp một auto-completion cho các lựa chọn. Người dùng vẫn có thể cung cấp bất kỳ câu trả lời nào, cho dù có gợi ý auto-completion:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

Ngoài ra, bạn có thể truyền một closure làm tham số thứ hai cho phương thức `anticipate`. closure sẽ được gọi mỗi khi người dùng nhập một ký tự vào. closure phải chấp nhận một tham số string có chứa các ký tự nhập vào của người dùng và trả về một loạt các tùy chọn để tự động hoàn thành:

    $name = $this->anticipate('What is your address?', function ($input) {
        // Return auto-completion options...
    });

<a name="multiple-choice-questions"></a>
#### Multiple Choice Questions

Nếu bạn cần cung cấp cho người dùng một danh sách các lựa chọn khi hỏi một câu hỏi, thì bạn có thể sử dụng phương thức `choice`. Bạn có thể set giá trị mặc định cho phương thức này thông qua index của mảng, và nó sẽ được trả về nếu người dùng không chọn bất kỳ tuỳ chọn nào của bạn index này có thể được chỉ định qua tham số thứ ba:

    $name = $this->choice(
        'What is your name?',
        ['Taylor', 'Dayle'],
        $defaultIndex
    );

Ngoài ra, phương thức `choice` chấp nhận tham số thứ tư và tùy chọn thứ năm để xác định số lần thử tối đa và có cho phép chọn nhiều hay không:

    $name = $this->choice(
        'What is your name?',
        ['Taylor', 'Dayle'],
        $defaultIndex,
        $maxAttempts = null,
        $allowMultipleSelections = false
    );

<a name="writing-output"></a>
### Viết Output

Để gửi một output đến console, hãy sử dụng các phương thức `line`, `info`, `comment`, `question` và `error`. Mỗi phương thức này sẽ sử dụng một màu ANSI thích hợp cho mục đích của chúng. Ví dụ: Để hiển thị một thông tin chung cho người dùng. Thì thông thường, phương thức `info` sẽ hiển thị trong console dưới dạng màu xanh lá cây:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        // ...

        $this->info('The command was successful!');
    }

Để hiển thị một thông báo lỗi, sử dụng phương thức `error`. Thông báo lỗi đó sẽ được hiển thị màu đỏ:

    $this->error('Something went wrong!');

Bạn có thể sử dụng phương thức `line` để hiển thị đoạn text, không có màu:

    $this->line('Display this on the screen');

Bạn có thể sử dụng phương thức `newLine` để hiển thị một dòng trống:

    // Write a single blank line...
    $this->newLine();

    // Write three blank lines...
    $this->newLine(3);

<a name="tables"></a>
#### Tables

Phương thức `table` giúp bạn dễ dàng định dạng chính xác nhiều hàng / cột dữ liệu. Tất cả những gì bạn cần làm là cung cấp tên cột và dữ liệu cho bảng và Laravel sẽ
tự động tính toán chiều rộng và chiều cao thích hợp cho bảng của bạn:

    use App\Models\User;

    $this->table(
        ['Name', 'Email'],
        User::all(['name', 'email'])->toArray()
    );

<a name="progress-bars"></a>
#### Progress Bars

Đối với các tác vụ chạy dài, có thể bạn sẽ cần hiển thị một thanh tiến trình thông báo cho người dùng biết mức độ hoàn thành của tác vụ. Sử dụng phương thức `withProgressBar`, Laravel sẽ hiển thị một thanh tiến trình và tăng tiến trình đó thông qua mỗi lần lặp của một giá trị lặp nhất định:

    use App\Models\User;

    $users = $this->withProgressBar(User::all(), function ($user) {
        $this->performTask($user);
    });

Thỉnh thoảng, bạn có thể cần kiểm soát nhiều hơn đối với cách tăng của thanh tiến trình. Đầu tiên, định nghĩa tổng số các bước mà tiến trình sẽ lặp. Sau đó, tăng thanh tiến trình sau khi xử lý xong từng bước:

    $users = App\Models\User::all();

    $bar = $this->output->createProgressBar(count($users));

    $bar->start();

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

> {tip} Để biết các tùy chọn nâng cao, hãy xem [tài liệu component Symfony Progress Bar](https://symfony.com/doc/current/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Đăng ký Command

Tất cả các lệnh console của bạn được đăng ký trong class `App\Console\Kernel` của ứng dụng, là "console kernel" của ứng dụng của bạn. Trong phương thức `commands` của class này, bạn sẽ thấy một lệnh gọi đến phương thức` load` của kernel. Phương thức `load` này sẽ quét thư mục `app/Console/Commands` và đăng ký tất cả các command mà nó chứa với Artisan. Bạn thậm chí có thể thực hiện thêm các cuộc gọi bổ sung để quét thêm các thư mục khác cho các lệnh Artisan:

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/../Domain/Orders/Commands');

        // ...
    }

Nếu cần thiết, bạn có thể đăng ký các lệnh theo cách thủ công bằng cách thêm tên class của command vào thuộc tính `$commands` trong class `App\Console\Kernel` của bạn. Nếu thuộc tính này chưa được định nghĩa trên kernel của bạn, thì bạn nên tự định nghĩa nó. Khi Artisan khởi động, tất cả các lệnh được liệt kê trong thuộc tính này sẽ được resolve bằng [service container](/docs/{{version}}/container) và được đăng ký với Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## Chạy command bên ngoài CLI

Thỉnh thoảng bạn có thể muốn chạy một command Artisan bên ngoài CLI. Ví dụ: bạn có thể chạy một command Artisan từ route hoặc controller. Bạn có thể sử dụng phương thức `call` trên facade `Artisan` để thực hiện điều này. Phương thức `call` chấp nhận tên một command hoặc tên một class làm tham số đầu tiên và một mảng các tham số của command đó làm tham số thứ hai. Exit code sẽ được trả về:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function ($user) {
        $exitCode = Artisan::call('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        //
    });

Ngoài ra, bạn có thể truyền toàn bộ lệnh Artisan sang phương thức `call` dưới dạng một chuỗi:

    Artisan::call('mail:send 1 --queue=default');

<a name="passing-array-values"></a>
#### Passing Array Values

Nếu command của bạn định nghĩa một tùy chọn là một mảng, bạn có thể truyền một mảng các giá trị cho tùy chọn đó:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/mail', function () {
        $exitCode = Artisan::call('mail:send', [
            '--id' => [5, 13]
        ]);
    });

<a name="passing-boolean-values"></a>
#### Passing Boolean Values

Nếu bạn cần định nghĩa một giá trị cho một tùy chọn không nhận giá trị, chẳng hạn như một flag `--force` trong lệnh `migrate:refresh`, bạn có thể truyền `true` hoặc `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="queueing-artisan-commands"></a>
#### Queueing Artisan Commands

Sử dụng phương thức `queue` trên facade `Artisan`, bạn thậm chí có thể queue các lệnh Artisan để chúng được xử lý trong background bởi [queue worker](/docs/{{version}}/queues) của bạn. Trước khi sử dụng phương thức này, hãy đảm bảo là bạn đã cấu hình queue của bạn và đang chạy một queue listener:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function ($user) {
        Artisan::queue('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        //
    });

Sử dụng phương thức `onConnection` và `onQueue`, bạn có thể chỉ định kết nối hoặc queue nào mà lệnh Artisan sẽ được gửi tới:

    Artisan::queue('mail:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

<a name="calling-commands-from-other-commands"></a>
### Calling Commands From Other Commands

Thỉnh thoảng bạn có thể muốn gọi các lệnh khác từ một lệnh Artisan hiện có. Bạn có thể làm như vậy bằng cách sử dụng phương thức `call`. Phương thức `call` này nhận vào tên của command và một mảng các tham số của command đó:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('mail:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

Nếu bạn muốn gọi một command khác và xoá đi tất cả các output của nó, bạn có thể sử dụng phương thức `callSilent`. Phương thức `callSilent` có cùng cách khai báo với phương thức `call`:

    $this->callSilently('mail:send', [
        'user' => 1, '--queue' => 'default'
    ]);

<a name="signal-handling"></a>
## Xử lý tín hiệu

Component symfony console, cái mà hỗ trợ cho Artisan console, cho phép bạn chỉ định các tín hiệu (nếu có) mà command của bạn cần xử lý. Ví dụ: bạn có thể chỉ định là command của bạn cần xử lý các tín hiệu `SIGINT` và` SIGTERM`.

Để bắt đầu, bạn nên implement interface `Symfony\Component\Console\Command\SignalableCommandInterface` trong class command Artisan của bạn. Interface này sẽ yêu cầu bạn định nghĩa hai phương thức: `getSubscribedSignals` và `handleSignal`:

```php
<?php

use Symfony\Component\Console\Command\SignalableCommandInterface;

class StartServer extends Command implements SignalableCommandInterface
{
    // ...

    /**
     * Get the list of signals handled by the command.
     *
     * @return array
     */
    public function getSubscribedSignals(): array
    {
        return [SIGINT, SIGTERM];
    }

    /**
     * Handle an incoming signal.
     *
     * @param  int  $signal
     * @return void
     */
    public function handleSignal(int $signal): void
    {
        if ($signal === SIGINT) {
            $this->stopServer();

            return;
        }
    }
}
```

Như bạn có thể thấy, phương thức `getSubscribedSignals` sẽ trả về một mảng các tín hiệu mà lệnh của bạn có thể xử lý, trong khi phương thức `handleSignal` sẽ nhận tín hiệu và có thể phản hồi tương ứng.

<a name="stub-customization"></a>
## Stub Customization

Lệnh `make` của Artisan console sẽ được sử dụng để tạo nhiều class khác nhau, chẳng hạn như controller, job, migration và các bài test. Các class này được tạo ra bằng cách sử dụng các file "stub" được điền sẵn các giá trị dựa trên đầu vào mà bạn đưa vào. Tuy nhiên, thỉnh thoảng bạn có thể muốn thực hiện các thay đổi nhỏ đối với các file do Artisan tạo ra. Để thực hiện điều này, bạn có thể sử dụng lệnh `stub:publish` để export ra các stub cơ bản nhất để tùy chỉnh:

    php artisan stub:publish

Các file stub đã được export sẽ nằm trong thư mục `stubs` trong thư mục gốc của ứng dụng của bạn. Bất kỳ thay đổi nào mà bạn thực hiện đối với các file stub này sẽ được phản ánh khi bạn tạo các class tương ứng khi sử dụng lệnh Artisan `make`.

<a name="events"></a>
## Events

Artisan gửi ba event khi chạy các lệnh: `Illuminate\Console\Events\ArtisanStarting`, `Illuminate\Console\Events\CommandStarting`, và `Illuminate\Console\Events\CommandFinished`. Event `ArtisanStarting` được gửi ngay lập tức khi Artisan bắt đầu chạy. Tiếp theo, event `CommandStarting` sẽ được gửi ngay trước khi lệnh được chạy. Cuối cùng, event `CommandFinished` sẽ được gửi sau khi một lệnh đã chạy xong.
