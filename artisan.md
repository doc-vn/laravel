# Artisan Console

- [Giới thiệu](#introduction)
- [Viết Command](#writing-commands)
    - [Tạo Command](#generating-commands)
    - [Cấu trúc Command](#command-structure)
    - [Closure Command](#closure-commands)
- [Định nghĩa đầu vào](#defining-input-expectations)
    - [Tham số](#arguments)
    - [Tuỳ chọn](#options)
    - [Input Arrays](#input-arrays)
    - [Input mô tả](#input-descriptions)
- [Input và output của Command](#command-io)
    - [Lấy giá trị input](#retrieving-input)
    - [Hỏi giá trị input](#prompting-for-input)
    - [Viết đầu ra](#writing-output)
- [Đăng ký Command](#registering-commands)
- [Chạy command bên ngoài CLI](#programmatically-executing-commands)
    - [Gọi Command từ một Command khác](#calling-commands-from-other-commands)

<a name="introduction"></a>
## Giới thiệu

Artisan là giao diện dòng lệnh đi kèm với Laravel. Nó cung cấp một số lệnh hữu ích có thể hỗ trợ bạn trong khi bạn xây dựng application của bạn. Để xem danh sách tất cả các lệnh Artisan có sẵn, bạn có thể sử dụng lệnh `list`:

    php artisan list

Mỗi lệnh cũng chứa màn hình "help" hiển thị và mô tả các tùy chọn và tham số khả dụng của lệnh. Để xem màn hình trợ giúp, đặt `help` trước tên của command:

    php artisan help migrate

#### Laravel REPL

Tất cả các application của Laravel đều chứa Tinker, một REPL được cung cấp bởi package [PsySH](https://github.com/bobthecow/psysh). Tinker cho phép bạn tương tác với toàn bộ application Laravel của bạn trên dòng lệnh, bao gồm ORM Eloquent, job,event, v.v. Để vào môi trường Tinker, hãy chạy lệnh Artisan `tinker`:

    php artisan tinker

<a name="writing-commands"></a>
## Viết Commands

Ngoài các lệnh được cung cấp với Artisan, bạn cũng có thể xây dựng các lệnh tùy chỉnh của riêng bạn. Các lệnh thường được lưu trữ trong thư mục `app/Console/Commands`; tuy nhiên, bạn có thể thoải mái chọn vị trí lưu trữ của bạn miễn là các lệnh của bạn có thể load được bởi Composer.

<a name="generating-commands"></a>
### Tạo Commands

Để tạo một lệnh mới, sử dụng lệnh Artisan `make:command`. Lệnh này sẽ tạo một class lệnh mới trong thư mục `app/Console/Commands`. Đừng lo lắng nếu thư mục này không tồn tại trong application của bạn, vì nó sẽ được tạo vào lần đầu tiên khi bạn chạy lệnh Artisan `make:command`. Lệnh được tạo ra sẽ chứa mặc định các thuộc tính và phương thức mà có trên tất cả các lệnh:

    php artisan make:command SendEmails

<a name="command-structure"></a>
### Cấu trúc Command

Sau khi đã tạo lệnh của bạn, bạn nên điền vào các thuộc tính `signature` và `description` của class, nó sẽ được sử dụng khi hiển thị lệnh của bạn trên màn hình `list`. Phương thức `handle` sẽ được gọi khi lệnh của bạn được thực thi. Bạn có thể đặt logic lệnh của bạn vào trong phương thức này.

> {tip} Để code có thể tái sử dụng tốt hơn, cách tốt nhất là giữ cho các lệnh console của bạn được rõ ràng và hãy trì hoãn chúng để các application service hoàn thành nhiệm vụ của chúng. Trong ví dụ dưới đây, lưu ý rằng chúng ta sẽ inject một service class để thực hiện "công việc nặng" như việc gửi e-mail.

Chúng ta hãy xem ví dụ một lệnh. Lưu ý rằng chúng ta có thể đưa bất kỳ inject nào chúng ta muốn vào hàm constructor của lệnh. Laravel [service container](/docs/{{version}}/container) sẽ tự động inject tất cả các phụ thuộc được khai báo theo dạng kiểu trong hàm constructor:

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

Các lệnh dựa trên Closure cung cấp một sự thay thế để định nghĩa các lệnh console như các class. Giống như cách mà route Closure làm là một thay thế cho controller, hãy nghĩ về lệnh Closure như là một thay thế cho các class lệnh. Trong phương thức `commands` của file `app/Console/Kernel.php` của bạn, Laravel sẽ load file `routes/console.php`:

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

Mặc dù file này không định nghĩa các route HTTP, nhưng nó định nghĩa các điểm đầu vào trên console để vào application của bạn. Trong file này, bạn có thể định nghĩa tất cả các route dựa trên Closure của bạn bằng phương thức `Artisan::command`. Phương thức `command` chấp nhận hai tham số: [command signature](#defining-input-expectations) và Closure để nhận vào các tham số và các tùy chọn của lệnh:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

Closure được liên kết với instance command cơ bản, vì vậy bạn có toàn quyền truy cập vào tất cả các phương thức helper mà bạn thường có thể truy cập trên một class command đầy đủ.

#### Khai báo dạng kiểu phụ thuộc

Ngoài việc nhận vào các tham số và tùy chọn của lệnh, Closure command cũng có thể loại thêm phụ thuộc bổ sung theo dạng khai báo kiểu mà bạn muốn resolve từ [service container](/docs/{{version}}/container):

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### Closure Command Descriptions

Khi định nghĩa một lệnh dựa trên Closure, bạn có thể sử dụng phương thức `describe` để thêm mô tả vào lệnh. Mô tả này sẽ được hiển thị khi bạn chạy các lệnh `php artisan list` hoặc `php artisan help`:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## Định nghĩa đầu vào

Khi viết lệnh console, thường thu nhận dữ liệu đầu vào từ người dùng thông qua các tham số hoặc tùy chọn. Laravel làm cho nó rất thuận tiện để xác định đầu vào mà bạn mong đợi từ người dùng bằng cách sử dụng thuộc tính `signature` trên các lệnh của bạn. Thuộc tính `signature` cho phép bạn xác định tên, tham số và các tùy chọn cho lệnh theo một cú pháp đơn, dễ hiểu, giống như cú pháp trên route.

<a name="arguments"></a>
### Tham số

Tất cả các tham số và tùy chọn do người dùng cung cấp được wrap trong dấu ngoặc nhọn. Trong ví dụ sau, lệnh sẽ định nghĩa một tham số **bắt buộc**: `user`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

Bạn cũng có thể tạo tham số tùy chọn và định nghĩa giá trị mặc định cho tham số:

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>
### Tuỳ chọn

Tùy chọn, giống như tham số, là một dạng khác của input user. Các tùy chọn sẽ được tiền tố bởi hai dấu gạch nối (`--`) khi chúng được chỉ định trên dòng lệnh. Có hai loại tùy chọn: loại tùy chọn nhận giá trị và loại không nhận giá trị. Các tùy chọn không nhận giá trị đóng vai trò là "công tắc" boolean. Chúng ta hãy xem một ví dụ về loại tùy chọn này:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

Trong ví dụ này, công tắc `--queue` có thể được chỉ định khi gọi lệnh Artisan. Nếu công tắc `--queue` được thông qua, giá trị của tùy chọn sẽ là `true`. Nếu không, giá trị sẽ là `false`:

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### Options With Values

Tiếp theo, chúng ta hãy xem một tùy chọn nhận một giá trị. Nếu người dùng phải chỉ định một giá trị cho một tùy chọn, thì hậu tố tên tùy chọn có dấu `=`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

Trong ví dụ này, người dùng có thể pass một giá trị cho tùy chọn như sau:

    php artisan email:send 1 --queue=default

Bạn có thể gán giá trị mặc định cho các tùy chọn này bằng cách chỉ định giá trị mặc định sau tên tùy chọn. Nếu không có giá trị tùy chọn nào được người dùng chuyển qua, giá trị mặc định sẽ được sử dụng:

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### Option Shortcuts

Để gán một shortcut khi định nghĩa một tùy chọn, bạn có thể chỉ định nó phía trước tên tùy chọn và sử dụng một dấu | để tách shortcut khỏi toàn bộ tên tùy chọn:

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### Input Arrays

Nếu bạn muốn định nghĩa các tham số hoặc tùy chọn để nhận đầu vào mảng, bạn có thể sử dụng ký tự `*`. Đầu tiên, chúng ta hãy xem một ví dụ chỉ định một tham số mảng:

    email:send {user*}

Khi gọi phương thức này, các tham số `user` có thể được pass theo dòng lệnh. Ví dụ: lệnh sau sẽ set giá trị của `user` thành `['foo', 'bar']`:

    php artisan email:send foo bar

Khi định nghĩa một tùy chọn nhận một đầu vào mảng, mỗi giá trị tùy chọn được pass cho lệnh nên được thêm tiền tố với tên tùy chọn:

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### Input mô tả

Bạn có thể gán mô tả cho các đầu vào tham số và tùy chọn bằng cách tách tham số khỏi mô tả bằng dấu hai chấm. Nếu bạn cần thêm một chút chỗ để định nghĩa lệnh của mình, hãy định nghĩa trên nhiều dòng:

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

Trong khi lệnh của bạn đang thực thi, rõ ràng bạn sẽ cần truy cập vào các giá trị của các tham số và tùy chọn đã được khai báo trong lệnh của bạn. Để làm như vậy, bạn có thể sử dụng các phương thức `argument` và `option`:

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

Các tùy chọn có thể được lấy ra dễ dàng như các tham số bằng cách sử dụng phương thức `option`. Để lấy tất cả các tùy chọn dưới dạng một mảng, hãy gọi phương thức `option`:

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->options();

Nếu tham số hoặc tùy chọn không tồn tại, `null` sẽ được trả về.

<a name="prompting-for-input"></a>
### Hỏi giá trị input

Ngoài việc hiển thị output, bạn cũng có thể yêu cầu người dùng cung cấp đầu vào trong quá trình đang thực thi lệnh của bạn. Phương thức `ask` sẽ hỏi người dùng với câu hỏi đã cho, chấp nhận đầu vào của họ và sau đó trả lại đầu vào của người dùng cho lệnh của bạn:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

Phương thức `secret` tương tự như `ask`, nhưng đầu vào của người dùng sẽ không hiển thị với họ khi họ gõ vào console. Phương thức này hữu ích khi yêu cầu thông tin nhạy cảm như mật khẩu:

    $password = $this->secret('What is the password?');

#### Asking For Confirmation

Nếu bạn cần yêu cầu người dùng xác nhận đơn giản, bạn có thể sử dụng phương thức `confirm`. Mặc định, phương thức này sẽ trả về `false`. Tuy nhiên, nếu người dùng nhập `y` hoặc `yes` để đáp lại confirm, phương thức sẽ trả về `true`.

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### Auto-Completion

Phương thức `anticipate` có thể được sử dụng để cung cấp auto-completion cho các lựa chọn có thể. Người dùng vẫn có thể chọn bất kỳ câu trả lời nào, cho dù có gợi ý auto-completion:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### Multiple Choice Questions

Nếu bạn cần cung cấp cho người dùng một tập hợp các lựa chọn được định nghĩa trước, bạn có thể sử dụng phương thức `choice`. Bạn có thể set index của mảng cho giá trị mặc định sẽ được trả về nếu không có tùy chọn nào được chọn:

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);

<a name="writing-output"></a>
### Writing Output

Để gửi output đến console, hãy sử dụng các phương thức `line`, `info`, `comment`, `question` và `error`. Mỗi phương thức này sẽ sử dụng màu ANSI thích hợp cho mục đích của chúng. Ví dụ: hãy hiển thị một số thông tin chung cho người dùng. Thông thường, phương thức `info` sẽ hiển thị trong console dưới dạng văn bản màu xanh lá cây:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

Để hiển thị một thông báo lỗi, sử dụng phương thức `error`. Văn bản thông báo lỗi thường được hiển thị màu đỏ:

    $this->error('Something went wrong!');

Nếu bạn muốn hiển thị giao diện output đơn giản không màu, hãy sử dụng phương thức `line`:

    $this->line('Display this on the screen');

#### Table Layouts

Phương thức `table` giúp dễ dàng định dạng chính xác nhiều hàng hoặc cột dữ liệu. Chỉ cần truyền vào các header và hàng cho phương thức. Chiều rộng và chiều cao sẽ được tính toán linh hoạt dựa trên dữ liệu đã cho:

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### Progress Bars

Đối với các tác vụ chạy dài, có thể hữu ích để hiển thị một chỉ số tiến trình. Sử dụng đối tượng output, chúng ta có thể bắt đầu, tiến và dừng thanh tiến trình. Đầu tiên, xác định tổng số bước mà tiến trình sẽ lặp qua. Sau đó, tiến thanh tiến trình sau khi xử lý xong từng mục:

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

Để biết các tùy chọn nâng cao hơn, hãy xem [Tài liệu component Symfony Progress Bar](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Đăng ký Command

Bởi vì phương thức `load` được gọi trong phương thức `commands` trong console kernel của bạn, nên tất cả các lệnh trong thư mục `app/Console/Commands` sẽ tự động được đăng ký với Artisan. Trong thực tế, bạn có thể thoải mái thực hiện gọi thêm các phương thức `load` để quét các thư mục khác cho các lệnh Artisan:

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

Bạn cũng có thể tự đăng ký các lệnh bằng cách thêm tên class của nó vào thuộc tính `$commands` của file `app/Console/Kernel.php` của bạn. Khi Artisan khởi động, tất cả các lệnh được liệt kê trong thuộc tính này sẽ được resolve bằng [service container](/docs/{{version}}/container) và được đăng ký với Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## Chạy command bên ngoài CLI

Thỉnh thoảng bạn có thể muốn chạy một lệnh Artisan bên ngoài CLI. Ví dụ: bạn có thể gọi một lệnh Artisan từ route hoặc controller. Bạn có thể sử dụng phương thức `call` trên facade `Artisan` để thực hiện điều này. Phương thức `call` chấp nhận tên của lệnh làm tham số thứ nhất và một mảng các tham số lệnh làm tham số thứ hai. Exit code sẽ được trả về:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Sử dụng phương thức `queue` trên facade `Artisan`, bạn thậm chí có thể queue các lệnh Artisan để chúng được xử lý trong background bởi [queue workers](/docs/{{version}}/queues) của bạn. Trước khi sử dụng phương thức này, hãy đảm bảo bạn đã cấu hình queue của mình và đang chạy queue listener:

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Bạn cũng có thể chỉ định kết nối hoặc queue mà lệnh Artisan sẽ được gửi tới:

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

#### Passing Array Values

Nếu lệnh của bạn định nghĩa một tùy chọn nhận một mảng, bạn có thể chuyển một mảng các giá trị cho tùy chọn đó:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

#### Passing Boolean Values

Nếu bạn cần chỉ định giá trị của một tùy chọn không nhận giá trị, chẳng hạn như flag `--force` trong lệnh `migrate:refresh`, bạn nên pass `true` hoặc `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### Gọi Command từ một Command khác

Thỉnh thoảng bạn có thể muốn gọi các lệnh khác từ một lệnh Artisan hiện có. Bạn có thể làm như vậy bằng cách sử dụng phương thức `call`. Phương thức `call` này nhận tên lệnh và một mảng các tham số lệnh:

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

Nếu bạn muốn gọi một lệnh console khác và xoá tất cả output của nó, bạn có thể sử dụng phương thức `callSilent`. Phương thức `callSilent` có cùng signature với phương thức `call`:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
