# Console Tests

- [Giới thiệu](#introduction)
- [Expecting Input và Output](#expecting-input-and-output)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc đơn giản hóa cách kiểm tra HTTP, Laravel cung cấp một API đơn giản để kiểm tra application console theo dữ liệu người dùng.

<a name="expecting-input-and-output"></a>
## Expecting Input và Output

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

Bạn có thể kiểm tra lệnh này bằng cách sử dụng bài test dưới đây, bài test này sử dụng các phương thức `expectsQuestion`, `expectsOutput`, và `assertExitCode`:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function testConsoleCommand()
    {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you program in?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you program in PHP.')
             ->assertExitCode(0);
    }


