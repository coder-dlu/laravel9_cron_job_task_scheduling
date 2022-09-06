# laravel9_cron_job_task_scheduling
## 1:  Install Laravel 9
```Dockerfile
composer create-project laravel/laravel laravel9_cron_job_task_scheduling
```
#3 2: Create New Command
```Dockerfile
php artisan make:command DemoCron --command=demo:cron
```
- Bây giờ thực hiện một số thay đổi trên tệp lệnh.
```Dockerfile
<?php
  
namespace App\Console\Commands;
  
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Http;
use App\Models\User;
  
class DemoCron extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'demo:cron';
  
    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Command description';
  
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
     * @return int
     */
    public function handle()
    {
        info("Cron Job running at ". now());
  
        /*------------------------------------------
        --------------------------------------------
        Write Your Logic Here....
        I am getting users and create new users if not exist....
        --------------------------------------------
        --------------------------------------------*/
        $response = Http::get('https://jsonplaceholder.typicode.com/users');
      
        $users = $response->json();
  
        if (!empty($users)) {
            foreach ($users as $key => $user) {
                if(!User::where('email', $user['email'])->exists() ){
                    User::create([
                        'name' => $user['name'],
                        'email' => $user['email'],
                        'password' => bcrypt('123456789')
                    ]);
                }
            }
        }
  
        return 0;
    }
}
```
## 3: Register as Task Scheduler
|               A                  |                        B                           |
| :-------------------------------:|:--------------------------------------------------:|
| ->everyMinute();                 |  Run the task every minute                         |
| ->everyFiveMinutes();            |  Run the task every five minutes                   |
| ->everyTenMinutes();             |  Run the task every ten minutes                    |
| ->everyFifteenMinutes();         |  Run the task every fifteen minutes                |
| ->everyThirtyMinutes();          |  Run the task every thirty minutes                 |
| ->hourly();                      |  Run the task every hour                           |  
| ->hourlyAt(17);                  |  Run the task every hour at 17 mins past the hour  |
| ->daily();                       |  Run the task every day at midnight                |
| ->dailyAt(’13:00′);              |  Run the task every day at 13:00                   |
| ->twiceDaily(1, 13);             |  Run the task daily at 1:00 & 13:00                |
| ->weekly();                      |  Run the task every week                           |
| ->weeklyOn(1, ‘8:00’);           |  Run the task every week on Tuesday at 8:00        |  
| ->monthly();                     |  Run the task every month                          |
| ->monthlyOn(4, ’15:00′);         |  Run the task every month on the 4th at 15:00      |
| ->quarterly();                   |  Run the task every quarter                        |
| ->yearly();                      |  Run the task every year                           |
| ->timezone(‘America/New_York’);  |  Set the timezone                                  |  

- Vào app/Console/Kernel.php
```Dockerfile
<?php
  
namespace App\Console;
  
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
  
class Kernel extends ConsoleKernel
{
    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('demo:cron')
                 ->everyMinute();
    } 
  
    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
   
        require base_path('routes/console.php');
    }
}
```
## 4: Run Scheduler Command For Test
```Dockerfile
php artisan schedule:run
```
- storage/logs/laravel.php
```Dockerfile
[2022-03-01 03:51:02] local.INFO: Cron Job running at 2022-03-01 03:51:02  
[2022-03-01 03:52:01] local.INFO: Cron Job running at 2022-03-01 03:52:01  
[2022-03-01 03:53:02] local.INFO: Cron Job running at 2022-03-01 03:53:02 
```
## 5:  Cron Job Setup on Server
```Dockerfile
crontab -e
```
- Bây giờ, thêm dòng dưới đây vào tệp crontab. đảm bảo rằng bạn cần đặt đường dẫn dự án của mình một cách chính xác trên đó
```Dockerfile
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```
