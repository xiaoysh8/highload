Fault tolerance and its implementation in PHP

When developing a web application, it's important to remember about fault tolerance. Fault tolerance - a design method in which the inoperability of a single element or application function does not disable the application itself, the program's resistance to failures. Do not confuse fault tolerance with a failover ( failover ) by switching failover functionality to a standby component. fault tolerant app

Strategy

Fault-tolerant architecture is ready for failure. The application is designed in such a way that when the components fall, they restart themselves without failures, losses or errors.

As an example, we use a script to send mail to the database. Not a fault-tolerant program in the event of a failure in execution will start the process first, if you restore the work. fault tolerant mailing script

A script written with regard to fault tolerance will understand that the last execution failed and will continue to send messages from the user on which the error occurred.

The script itself looks like this:

<?

# получаем id пользователя для восстановления, если скрипт был завершен некорректно
$restart_from = file_get_contents('/tmp/mail.status');


# получаем список всех пользователей
$users = get_list('SELET * FROM users ORDER BY id');

foreach ( $users as $user )
{
    # пропускаем отправку, если переменная восстановления задана
    if ( $restart_from && $user['id'] <= $restart_from ) continue;

    send_email($user['id']);
    
    # сохраняем id пользователя в файл статуса
    file_put_contents('/tmp/mail.status', $user['id']);
}

# удаляем файл статуса 
unlink('/tmp/mail.status');
# The absence of a status file indicates the successful completion of the previous sending

Another option for a fault-tolerant application is to run scripts in separate child processes, using multithreading. If malfunctions occur, the process will end with an error, and the vorker will restart. Gearman is suitable for this .

Signals

In UNIX-like systems, signals are a method of notifying processes about events. They are executed asynchronously and with high priority, so when receiving such notification, the system interrupts the execution of the process. UNIX signals

What does the signals in the article on fault tolerance mean? The function is suitable for stopping or restarting processes (daemons) in case of an error, and the task is correctly completed.

PCNTL

For PHP, the pcntl process manager extension is available. It uses a UNIX-style control of processes and subroutines, including signal processing. So on its basis the simplest fault-tolerant system is implemented:

<?
declare(ticks = 1);
class SIG { public static $cought = 0; }
…

function sig_handler($sig) {
    SIG::$cought = $sig;
    echo 'Finishing all operations to exit...' . "\n";
}

pcntl_signal(SIGINT,  "sig_handler");
pcntl_signal(SIGTERM, "sig_handler");
pcntl_signal(SIGHUP,  "sig_handler");


while ( true )
{
   /*
      Длинный код скрипта для обработки с нужными действиями
   */
…

    if ( SIG::$cought )
    {
        echo 'finished operations, exiting now' . "\n";
        exit;
    }
}
# It catches signals SIGINT, SIGTERM, SIGHUP, if such a signal comes, then the script stops

This approach is used to correctly restart the scripts when using the supervisor .

pcntl is suitable for PHP asynchronous execution as an alternative to pthreads . The difference between extensions is that pcntl is a process manager, and pthreads is a thread manager.

Phystrix

PHP-library Phystrix was developed under the impression and on the principle of Netflix Hystrix. It is designed for distributed applications with dependencies.Phystrix scheme

At its core, Phystrix is ​​a wrapper for PHP scripts that isolates their execution from each other, providing fault tolerance and no cascading failures.

Principle of operation

When you create the Phystrix command, you use the run () method , which performs the required function (database query, API call), and the getFallback () method , which calls the default value.

Then, when you call execute (), Phystrix starts the function denoted by run () . And in case of an error, it calls the default value in getFallback () , for example, "The function is temporarily unavailable."

For the library to work correctly, an APC extension is required , and for installation it is recommended to use Composer :

"require": {
     "odesk/phystrix": "dev-master"
}
# Will use the company's repository on GitHub

In the simplest case, an example of using Phystrix would look like this:

use Odesk\Phystrix\AbstractCommand;

class MyCommand extends AbstractCommand
{
    protected $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    /**
     # Внутренний вызов функции только если запрос разрешен
     # @return mixed
     */
    protected function run()
    {
        return 'Hello ' . $this->name;
    }
}
# All commands should use AbstractCommand inheritance

And then the command can be executed using the execute () method:

$myCommand = $phystrix->getCommand('MyCommand', 'Alex'); # 'Alex' передается конструктору MyCommand
$result = $myCommand->execute();
# Additional parameters in getCommand are passed to the constructor

Example of using Phystrix

The library can be used for many functions, HTTP requests, custom or standard APIs, and also SQL:

use Odesk\Phystrix\AbstractCommand;
    
class GetTop extends AbstractCommand
{
    protected $startDate;
   
    public function __construct($startDate)
    {
        $this->startDate = $startDate;
    }
  
     protected function run()
    {
        # получение зависимостей DB из служебного локатора Phystrix
        $db = $this->serviceLocator->get('db');
        $sql = "
            SELECT
             U.DisplayName,
             COUNT(C.ID) AS CommentCount
            FROM
             Users AS U
             INNER JOIN Comments AS C ON U.ID = C.UserID
            GROUP BY
             U.DisplayName
            WHERE C.time > ?
            ORDER BY
             COUNT(C.ID) DESC
            LIMIT 1;
        ";
        return $db->fetchOne($sql, $this->_startDate); 
    }
      
    protected function getFallback()
    {
        return "функция временно недоступна";
    }
}
# The SQL query is wrapped in the design pattern

Well and for start all the same method execute () is used:

$getTopCommenterNameCmd =
    $phystrixFactory->getCommand('GetTop', strtotime('-1 week'));
$topCommenterName = $getTop->execute();
# Phystrix will independently check all dependencies

The most important thing

Do not neglect the fault tolerance of the service or application. PHP signals are sufficient to create a fault-tolerant architecture. And external libraries and solutions, like Phystrix, will provide flexibility, but will bring additional complexity when setting up and implementing.