# Console Tests

- [Giới thiệu](#introduction)
- [Kỳ vọng thành công hay thất bai](#success-failure-expectations)
- [Kỳ vọng input và output](#input-output-expectations)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc đơn giản hóa cách kiểm tra HTTP, Laravel cung cấp một API đơn giản để kiểm tra [các lệnh console tuỳ chỉnh](/docs/{{version}}/artisan) của application của bạn.

<a name="success-failure-expectations"></a>
## Kỳ vọng thành công hay thất bai

Để bắt đầu, hãy khám phá cách kiểm tra liên quan đến exit code của lệnh Artisan. Để thực hiện điều này, chúng ta sẽ sử dụng phương thức `artisan` để gọi một lệnh Artisan từ bài test của chúng ta. Sau đó, chúng ta sẽ sử dụng phương thức `assertExitCode` để kiểm tra xem lệnh đã chạy xong với một exit code nhất định:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function test_console_command()
    {
        $this->artisan('inspire')->assertExitCode(0);
    }

Bạn có thể sử dụng phương thức `assertNotExitCode` để kiểm tra rằng lệnh đã thoát với một exit code nhất định:

    $this->artisan('inspire')->assertNotExitCode(1);

Tất nhiên, tất cả các lệnh terminal thường thoát với một status code là `0` khi chúng thành công và exit code khác 0 khi chúng thất bại. Do đó, để thuận tiện, bạn có thể sử dụng kiểm tra `assertSuccessful` và `assertFailed` để kiểm tra rằng một lệnh nhất định đã thoát với exit code thành công hay không:

    $this->artisan('inspire')->assertSuccessful();

    $this->artisan('inspire')->assertFailed();

<a name="input-output-expectations"></a>
## Kỳ vọng input và output

Laravel cho phép bạn dễ dàng "mô phỏng" cách nhập của người dùng trên các cửa sổ dòng lệnh bằng phương thức `expectsQuestion`. Ngoài ra, bạn cũng có thể sử dụng các phương thức `assertExitCode` và `expectsOutput` để chỉ định các exit code và các text mà bạn mong muốn được xuất hiện trên cửa sổ dòng lệnh. Ví dụ: hãy xem lệnh console sau:

    Artisan::command('question', function () {
        $name = $this->ask('What is your name?');

        $language = $this->choice('Which language do you program in?', [
            'PHP',
            'Ruby',
            'Python',
        ]);

        $this->line('Your name is '.$name.' and you program in '.$language.'.');
    });

Bạn có thể kiểm tra lệnh này bằng cách sử dụng bài test dưới đây, bài test này sử dụng các phương thức `expectsQuestion`, `expectsOutput`, `doesntExpectOutput`, `expectsOutputToContain`, `doesntExpectOutputToContain`, và `assertExitCode`:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function test_console_command()
    {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you prefer?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
             ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
             ->expectsOutputToContain('Taylor Otwell')
             ->doesntExpectOutputToContain('you prefer Ruby')
             ->assertExitCode(0);
    }

<a name="confirmation-expectations"></a>
#### Confirmation Expectations

Khi viết một lệnh để kiểm tra một confirmation dưới dạng câu trả lời "có" hoặc "không", bạn có thể sử dụng phương thức `expectsConfirmation`:

    $this->artisan('module:import')
        ->expectsConfirmation('Do you really wish to run this command?', 'no')
        ->assertExitCode(1);

<a name="table-expectations"></a>
#### Table Expectations

Nếu lệnh của bạn hiển thị một bảng thông tin bằng cách sử dụng phương thức `table` của Artisan, thì việc viết các kỳ vọng output cho toàn bộ bảng có thể rất phức tạp. Thay vào đó, bạn có thể sử dụng phương thức `expectsTable`. Phương thức này sẽ chấp nhận header của bảng làm tham số đầu tiên và dữ liệu của bảng đó làm tham số thứ hai:

    $this->artisan('users:all')
        ->expectsTable([
            'ID',
            'Email',
        ], [
            [1, 'taylor@example.com'],
            [2, 'abigail@example.com'],
        ]);
