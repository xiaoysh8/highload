Queues for Gearman and PHP

Gearman is a simple queuing system . Has a lot of clients, including PHP. Allows you to scale to multiple servers, and also has the ability to prioritize tasks.

In-app implementation

Let's build a simple solution for asynchronously sending mail in the application using Gearman and PHP. The solution should include a server, a client and a vendor: Gearman PHP queue

The server is Gearman itself, which receives messages from the client
The client is the main PHP application that sends a message to the server
Worker is a PHP script that receives a message from the task server and performs some actions
Message

What is a message? A message is any information that a client sends to a queue server. Then the same information is sent to the handler (vorker). If we want to transfer the sending of mail to the queue system, we will need to send the following message:

address of the recipient
body of the letter
message header
Server

For Debian, the installation is very simple:

apt-get install gearman-job-server php-gearman
After installation, you must start the server:

/etc/init.d/gearmand start
Client

In our client (the main application) sending email messages will work through the queue system. So instead of actually sending a letter, send a message to Gearman:

<?
$mail = array(
  'to' => 'test@gmail.com',
  'subject' => 'Привет',
  'body' => 'Это тестовое сообщение',
);


# Подключаемся к серверу Gearman
$client = new GearmanClient();
$client->addServer('127.0.0.1', '4730');


# Шлем сообщение
$client->doBackground('sendmail', json_encode($mail));
# sendmail is the type of the task (choose arbitrarily), $ mail - data for the letter

Note that you need to use json_encode () , because the client only accepts text values. The doBackground () method is the most important part of the work. This method sends a message with the transmitted data to the Gearman server.

Task Handler

The worker is a separate PHP script that constantly checks the server for new tasks. As soon as the task comes - it performs the logic connected with it (in our case this will be the actual delivery of the letter). Let's create a worker.php:

<?
$worker = new GearmanWorker();
$worker->addServer();

$worker->addFunction('sendmail', 'send_mail');

while (1)
{
  $worker->work();
  if ($worker->returnCode() != GEARMAN_SUCCESS) break;
}

function send_mail($job)
{
  $workload = $job->workload();
  $data = json_decode($workload, true);

  mail($data['to'], $data['subject'], $data['body']);
}
# This vorker will check messages on the server type "sendmail", and for their processing will use the send_mail () function , announced below.

Next, we just run this script on the command line, so that it runs constantly:

php worker.php &
More details about how to run the background process in PHP .

Prioritization

Gearman supports the prioritization of tasks. This allows you to perform important tasks faster if there are multiple messages in the queue. Priorities are used as follows:

<?
$client->doHighBackground ('sendmail', json_encode($mail));
# high priority, will be processed first

<?
$client->doLowBackground ('sendmail', json_encode($mail));
# low priority, will be processed last

Scalability and Fault Tolerance

Gearman supports work with multiple servers, This will allow to expand rapidly with increasing load. To do this, run several Gearman servers, and in the application connect to them this way:

<?
$client->addServers("10.0.0.1:4730,10.0.0.2:4730");
The most important

Gearman is a very simple and reliable solution for implementing asynchronous operation of slow parts of the application. Built-in support for scaling will allow you to use it for systems with high load.