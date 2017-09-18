Queues at Beanstalk

At its core Beanstalkd is a simplified and easy queuing system that was developed to fit Causes. It is represented as a task manager for a distributed application that collects pending tasks (sending mail, various kinds of requests).Beanstalkd scheme

Certain processes allocate tasks to the queue, and the service providers receive and execute tasks from the queue.

Among the basic features of beanstalkd are :

prioritization;
distribution - several servers beanstalkd work on the principle of Memcached (can be scaled);
deferred execution (t.ch. bury) for subsequent execution;
external libraries (PHP, Python and many others), the web console;
timeout for automatic queue.
Terms and commands of beanstalkd

The terminology of the tool differs from the usual messaging servers, the same Gearman , for example:

jobs - the same as messages;
tubes - queues in which messages are placed for transfer to workers (workers);
producers - applications that create tasks (or messages);
consumers are applications that receive jobs from the queue for processing.
There are also a few basic commands for beanstalkd:

put - add a new job to the queue (including deferred);
reserve - get the job from the queue;
delete - delete the job after execution;
bury - postpone the task after completion;
kick - pull the job from the bury status and put it in the queue;
release - to set the status "ready" for the task after execution, the id and priority are saved.
beanstalkd queue
More commands and options on the official page of the system .

Installation and configuration

Beanstalkd is included in the aptitude package system :

sudo aptitude install beanstalkd
# Also compiled from sources with GitHub

And to run it is enough to perform:

beanstalkd
# By default, it is executed locally using port 11300

After installing the daemon, beanstalkd can also be managed as an OS service. It runs in RAM, and to provide fault tolerance it starts with an additional option:

beanstalkd -b ~/beanstore &
# Write the queue data in the beanstore directory, run in the background

If you restart the beanstalkd with the same parameters, it will first check the log and continue execution from the point of stop.

The beanstalkd daemon can be run with the following options:

-b DIR   директория для валидации
 -f MS    fsync каждое значение миллисекунд ( -f0 для "always fsync")
 -F       без fsync (по умолчанию)
 -l ADDR  слушать адрес (0.0.0.0 по умолчанию)
 -p PORT  слушать порт (11300 по умолчанию)
 -u USER  стать пользователем и группой
 -z BYTES максимальный размер задания в байтах (65535 по умолчанию)
 -s BYTES размер каждого файла валидации (10485760 по умолчанию)
            (будут округлены до значения, кратного 512 байт)
 -c       уменьшить binlog (по умолчанию)
 -n       не уменьшать binlog
 -v       показать версию
 -V       улучшенный вывод
 -h       показать справку
# Options can be combined

In addition, beanstalkd supports clients for all popular languages and has an API for creating proprietary clients.

Example of using Ruby

For the first example, use the Beaneater client for Ruby. The location of the task (message) in the queue will look like this:

require 'beaneater'
require 'json'

beanstalk = Beaneater::Pool.new(['localhost:11300'])
tube = beanstalkd.tubes['my-tube']
job = {some: 'key', value: 'object'}.to_json

tube.put job
# Specifies the address and port, the name of the queue and the message itself, in the JSON format

After that, you need to create a script to catch the tasks from the queue:

beanstalkd.tubes.watch!('my-tube')
loop do
  job = beanstalk.tubes.reserve
  begin
    # обработка задачи
    job.delete
  rescue Exception => e
    job.bury
  end
end
# Waits for the task to be ready, processes it and repeats the whole process

To manage application-consumers it is logical to use process controllers, like supervisord .

If a large, heavy task is being handled, the upper example can be represented as:

beanstalkd.jobs.register('my-tube') do |job|
  # ... обработка задачи
end

beanstalkd.jobs.process!
# "Turns" the script, backing up, processing, and then deleting or postponing the task, based on the result

Example of using PHP

The second example implements the sending of mail using Mandrill . For the convenience of installing and using the Pheanstalk client for PHP, it is recommended to use Composer :

composer require mandrill/mandrill pda/pheanstalk
# Setting dependencies

It will take only two scripts: the task provider to the queue and the consumer who will handle the jobs.

The vendor code will look like:

<?php

require_once __DIR__ . '/vendor/autoload.php';

$email = array(
    'to' => 'xyz@example.com',
    'from' => 'abc@example.com',
    'subject' => 'Subject',
    'body' => 'Some text'
);

$pheanstalk = new \Pheanstalk\Pheanstalk('127.0.0.1');


# Добавляет JSON для задачи "email_queue"
$pheanstalk
    ->useTube('email_queue')
    ->put(json_encode($email));
# Creates a local task email_queue - the JSON string of the array with data

A vorker will look like this:

<?php

require_once __DIR__ . '/vendor/autoload.php';

$pheanstalk = new \Pheanstalk\Pheanstalk('127.0.0.1');


# Чтение очереди beanstalkd
while (true) {
    # Проверка соединения
    if (!$pheanstalk->getConnection()->isServiceListening()) {
        echo "Ошибка соединения, подождите... \n";

        # Ждет 5 с
        sleep(5);

        # Запуск следующей итерации
        continue;
    }

    # Получить задачу из очереди, если она готова
    $job = $pheanstalk
        ->watch('email_queue')
        ->ignore('default')
        ->reserve();

    $email = json_decode($job->getData(), true);

    try {
         # Отправка почты с использованием Mandrill API
        $mandrill = new Mandrill('[MANDRILL-API-KEY');

        $message = array(
            'html' => $email['body'],
            'subject' => $email['subject'],
            'from_email' => $email['from'],
            'to' => array(
                array(
                    'email' => $email['to'],
                )
            )
        );

        # Непосредственная отправка письма
        $result = $mandrill->messages->send($message);

       # Удалить задачу из очереди
        $pheanstalk->delete($job);

        echo "Письмо отправлено \n";

    } catch (Exception $e) {
        echo "Ошибка отправки - {$e->getMessage()} \n";
    }
}
# Do not forget to use your unique API

Then you can run the command:

nohup php email_worker.php > path/to/logfile.txt &
# Ignores the SIGHUP signal, writes data to a text file, runs in the background

The most important thing

Beanstalkd is suitable for implementing a queue system of any complexity in any popular programming language. And distributed execution will allow using the tool in highly-loaded applications.