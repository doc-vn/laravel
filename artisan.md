# Artisan Console

- [Giới thiệu](#introduction)
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

<a name="introduction"></a>
## Giới thiệu

Artisan là một giao diện dòng lệnh đi kèm với Laravel. Nó cung cấp một số lệnh hữu ích có thể hỗ trợ bạn trong khi bạn xây dựng application. Để xem danh sách tất cả các lệnh Artisan có sẵn, bạn có thể sử dụng lệnh `list`:

    php artisan list

Mỗi lệnh cũng chứa một lệnh "help" để hiển thị và mô tả các tùy chọn và các tham số dành cho lệnh đó. Để xem lệnh help, hãy set `help` vào trước tên của command:

    php artisan help migrate

#### Laravel REPL

Tất cả các application của Laravel đều chứa Tinker, một REPL cung cấp bởi package [PsySH](https://github.com/bobthecow/psysh). Tinker cho phép bạn tương tác trực tiếp với toàn bộ application Laravel của bạn trên command line, bao gồm ORM Eloquent, job, event, vv... Để vào được môi trường Tinker, hãy chạy lệnh Artisan `tinker`:

    php artisan tinker

<a name="writing-commands"></a>
## Viết Commands

Ngoài các lệnh được cung cấp với Artisan, bạn cũng có thể tự xây dựng các lệnh của riêng bạn. Các lệnh thường được lưu trữ trong thư mục `app/Console/Commands`; tuy nhiên, bạn cũng có thể thoải mái chọn vị trí lưu trữ mà bạn muốn, miễn là các lệnh của bạn có thể load được bởi Composer.

<a name="generating-commands"></a>
### Tạo Commands

Để tạo một lệnh mới, sử dụng lệnh Artisan `make:command`. Lệnh này sẽ tạo một class command mới trong thư mục `app/Console/Commands`. Đừng lo lắng nếu thư mục này không tồn tại trong application của bạn, vì nó sẽ được tạo vào lần đầu tiên bạn chạy lệnh Artisan `make:command`. Command được tạo ra sẽ có mặc định các thuộc tính và các phương thức mà đều có trên mỗi command:

    php artisan make:command SendEmails

<a name="command-structure"></a>
### Cấu trúc Command

Sau khi đã tạo xong command, bạn hãy thay đổi các thuộc tính `signature` và `description`, command sẽ được sử dụng các thuộc tính đó để hiển thị thông tin command của bạn trên màn hình `list`. Phương thức `handle` sẽ được gọi khi lệnh của bạn được thực thi. Bạn có thể cài đặt logic của bạn vào trong phương thức này.

> {tip} Để code của bạn có thể tái sử dụng tốt hơn, thì cách tốt nhất là giữ cho các command của bạn được "nhẹ" và hãy để các application service hoàn thành nhiệm vụ đó cho bạn. Trong ví dụ dưới đây, hãy chú ý rằng chúng ta sẽ inject một service class để thực hiện một "công việc nặng" như việc gửi e-mail.

Chúng ta hãy xem một ví dụ về command. Lưu ý rằng chúng ta có thể inject bất kỳ service nào mà chúng ta muốn vào hàm constructor của command. Laravel [service container](/docs/{{version}}/container) sẽ tự động inject tất cả các phụ thuộc mà được khai báo trong hàm constructor:

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
### Closure Command

Các command được tạo dựa trên closure sẽ cung cấp thêm một giải pháp để định nghĩa các command. Giống như cách mà các closure route làm, là tạo thêm một cách định nghĩa cho controller, bạn hãy nghĩ các closure command này như là một cách định nghĩa khác cho các class command, thay vì phải tạo ra một file command mới. Trong phương thức `commands` ở trong file `app/Console/Kernel.php` của bạn, Laravel sẽ load sẵn file `routes/console.php`:

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

Mặc dù file này không định nghĩa các HTTP route, nhưng nó định nghĩa các closure dựa theo format của route vào trong application của bạn. Trong file này, bạn có thể định nghĩa tất cả các closure dựa trên route của bạn bằng phương thức `Artisan::command`. Phương thức `command` chấp nhận hai tham số: một là một [command signature](#defining-input-expectations) và hai là một closure để nhận vào các tham số và các option của command:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

Closure sẽ được liên kết với một instance command cơ bản, nên bạn có toàn quyền truy cập vào tất cả các phương thức helper mà bạn thường dùng trên một class command cơ bản.

#### Khai báo dạng kiểu phụ thuộc

Ngoài việc nhận vào các tham số và các option của command, Closure command cũng có thể khai báo thêm các phụ thuộc mà bạn muốn resolve từ [service container](/docs/{{version}}/container):

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### Closure Command Descriptions

Khi định nghĩa một command dựa trên Closure, bạn có thể sử dụng phương thức `describe` để thêm mô tả cho command. Mô tả này sẽ được hiển thị khi bạn chạy lệnh `php artisan list` hoặc lệnh `php artisan help`:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## Định nghĩa Input

Khi viết một lệnh console, thường thu nhận các dữ liệu đầu vào từ người dùng thông qua các tham số hoặc các option. Laravel làm cho nó rất thuận tiện để xác định đầu vào mà bạn mong muốn từ người dùng bằng cách sử dụng thuộc tính `signature` trên mỗi lệnh của bạn. Thuộc tính `signature` cho phép bạn định nghĩa tên, tham số và các option cho lệnh theo một cú pháp đơn giản, dễ hiểu, giống như cú pháp trên route.

<a name="arguments"></a>
### Tham số

Tất cả các tham số và các tùy chọn do người dùng cung cấp được wrap trong một dấu ngoặc nhọn. Trong ví dụ sau, lệnh sẽ định nghĩa một tham số **bắt buộc**: `user`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

Bạn cũng có thể tạo ra tham số tùy chọn và định nghĩa giá trị mặc định cho các tham số đó:

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>
### Tuỳ chọn

Tùy chọn, giống như một tham số, là một dạng khác của input user. Các tùy chọn sẽ được gán tiền tố với hai dấu gạch nối (`--`) khi chúng được định nghĩa trên dòng lệnh. Có hai loại tùy chọn: loại tùy chọn nhận một giá trị và loại tuỳ chọn không nhận giá trị nào. Các tùy chọn không nhận giá trị đóng vai trò như là một "switch" boolean. Chúng ta hãy xem một ví dụ về loại tùy chọn này:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

Trong ví dụ này, switch `--queue` có thể được chỉ định khi gọi lệnh Artisan. Nếu switch `--queue` được thông qua, giá trị của tùy chọn sẽ là `true`. Nếu không, giá trị sẽ là `false`:

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### Tuỳ chọn với giá trị

Tiếp theo, chúng ta hãy xem một tùy chọn nhận một giá trị. Nếu người dùng phải chỉ định một giá trị cho một tùy chọn, thì hãy thêm hậu tố vào tên của tùy chọn đó bằng dấu `=`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

Trong ví dụ này, người dùng có thể truyền một giá trị cho tùy chọn đó như sau:

    php artisan email:send 1 --queue=default

Bạn cũng có thể gán một giá trị mặc định cho các tùy chọn này bằng cách chỉ định giá trị mặc định sau tên mỗi tùy chọn. Nếu không có giá trị tùy chọn nào được người dùng truyền vào, thì giá trị mặc định sẽ được sử dụng:

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### Option Shortcuts

Để gán một shortcut khi định nghĩa một tùy chọn, bạn có thể chỉ định nó vào phía trước tên của một tùy chọn và sử dụng một dấu | để tách shortcut khỏi toàn bộ tên tùy chọn:

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### Input cho một mảng

Nếu bạn muốn định nghĩa các tham số hoặc tùy chọn để nhận vào một mảng, bạn có thể sử dụng ký tự `*`. Đầu tiên, chúng ta hãy xem một ví dụ định nghĩa một tham số là một mảng:

    email:send {user*}

Khi gọi phương thức này, các tham số `user` có thể được truyền theo dòng lệnh. Ví dụ: lệnh sau sẽ set giá trị của `user` thành `['foo', 'bar']`:

    php artisan email:send foo bar

Khi định nghĩa một tùy chọn nhận vào một mảng, thì mỗi giá trị tùy chọn được truyền vào cho lệnh nên thêm một tiền tố cho mỗi tên tùy chọn:

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### Thêm mô tả cho Input

Bạn có thể gán một mô tả cho các input đầu vào như tham số hoặc tùy chọn bằng cách tách tham số đó ra khỏi mô tả bằng dấu hai chấm. Nếu bạn cần thêm một chút chỗ trống để định nghĩa thêm cho lệnh của mình, hãy định nghĩa nó trên nhiều dòng:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## Input và output của Command

<a name="retrieving-input"></a>
### Lấy giá trị input

Trong khi lệnh của bạn đang thực thi, rõ ràng bạn sẽ cần truy cập vào các giá trị của các tham số và các tùy chọn đã được khai báo trong lệnh của bạn. Để làm như vậy, bạn có thể sử dụng các phương thức `argument` và `option`:

    /**
     * Execute the console command.
     *
     * @return mixed
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

    // Retrieve all options...
    $options = $this->options();

Nếu tham số hoặc tùy chọn không tồn tại, `null` sẽ được trả về.

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

#### Xác nhận

Nếu bạn cần yêu cầu người dùng xác nhận, bạn có thể sử dụng phương thức `confirm`. Mặc định, phương thức này sẽ trả về `false`. Tuy nhiên, nếu người dùng nhập `y` hoặc `yes` để trả lời confirm, phương thức sẽ trả về `true`.

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### Auto-Completion

Phương thức `anticipate` có thể được sử dụng để cung cấp một auto-completion cho các lựa chọn. Người dùng vẫn có thể chọn bất kỳ câu trả lời nào, cho dù có gợi ý auto-completion:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### Multiple Choice Questions

Nếu bạn cần cung cấp cho người dùng một danh sách các lựa chọn để người dùng chọn, thì bạn có thể sử dụng phương thức `choice`. Bạn có thể set giá trị mặc định cho phương thức này thông qua index của mảng, và nó sẽ được trả về nếu người dùng không chọn bất kỳ tuỳ chọn nào của bạn:

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);

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
        $this->info('Display this on the screen');
    }

Để hiển thị một thông báo lỗi, sử dụng phương thức `error`. Thông báo lỗi đó sẽ được hiển thị màu đỏ:

    $this->error('Something went wrong!');

Nếu bạn muốn hiển thị giao diện output đơn giản không màu, hãy sử dụng phương thức `line`:

    $this->line('Display this on the screen');

#### Table Layouts

Phương thức `table` sẽ giúp bạn dễ dàng định dạng chính xác những dữ liệu mà có nhiều hàng hoặc nhiều cột. Chỉ cần truyền vào các tiêu đề và các dòng dữ liệu cho phương thức. Chiều rộng và chiều cao sẽ được tính toán linh hoạt dựa trên dữ liệu được đưa vào:

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### Progress Bars

Đối với các tác vụ chạy dài, bạn có thể cần hiển thị một tiến trình phần trăm. Sử dụng đối tượng output, chúng ta có thể bắt đầu, tiến và dừng thanh tiến trình. Đầu tiên, định nghĩa tổng số các bước mà tiến trình sẽ lặp. Sau đó, tiến thanh tiến trình sau khi xử lý xong từng bước:

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

Để biết các tùy chọn nâng cao, hãy xem [Tài liệu component Symfony Progress Bar](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Đăng ký Command

Bởi vì phương thức `load` đã được gọi trong phương thức `commands` trong console kernel của bạn, nên tất cả các command trong thư mục `app/Console/Commands` sẽ được tự động đăng ký với Artisan. Trong thực tế, bạn có thể thoải mái thực hiện gọi thêm các phương thức `load` để quét các thư mục khác cho các command Artisan mà bạn đã tạo:

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

Bạn cũng có thể tự đăng ký các command của bạn bằng cách thêm tên class của command đó vào thuộc tính `$commands` trong file `app/Console/Kernel.php`. Khi Artisan khởi động, tất cả các lệnh được liệt kê trong thuộc tính này sẽ được resolve bằng [service container](/docs/{{version}}/container) và được đăng ký với Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## Chạy command bên ngoài CLI

Thỉnh thoảng bạn có thể muốn chạy một command Artisan bên ngoài CLI. Ví dụ: bạn có thể chạy một command Artisan từ route hoặc controller. Bạn có thể sử dụng phương thức `call` trên facade `Artisan` để thực hiện điều này. Phương thức `call` chấp nhận tên của command làm tham số đầu tiên và một mảng các tham số của command đó làm tham số thứ hai. Exit code sẽ được trả về:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Sử dụng phương thức `queue` trên facade `Artisan`, bạn thậm chí có thể dùng queue cho các command Artisan để chúng được xử lý trong background, bởi [queue workers](/docs/{{version}}/queues) của bạn. Trước khi sử dụng phương thức này, hãy đảm bảo rằng bạn đã cấu hình queue và đang chạy queue listener:

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Bạn cũng có thể định nghĩa các kết nối hoặc queue mà các command Artisan sẽ được gửi tới:

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

#### Passing Array Values

Nếu command của bạn định nghĩa một tùy chọn là một mảng, bạn có thể truyền một mảng các giá trị cho tùy chọn đó:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

#### Passing Boolean Values

Nếu bạn cần định nghĩa một giá trị cho một tùy chọn không nhận giá trị, chẳng hạn như một flag `--force` trong lệnh `migrate:refresh`, bạn có thể truyền `true` hoặc `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### Gọi Command từ một Command khác

Thỉnh thoảng bạn có thể muốn gọi các lệnh khác từ một lệnh Artisan hiện có. Bạn có thể làm như vậy bằng cách sử dụng phương thức `call`. Phương thức `call` này nhận vào tên của command và một mảng các tham số của command đó:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

Nếu bạn muốn gọi một command khác và xoá đi tất cả các output của nó, bạn có thể sử dụng phương thức `callSilent`. Phương thức `callSilent` có cùng cách khai báo với phương thức `call`:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
