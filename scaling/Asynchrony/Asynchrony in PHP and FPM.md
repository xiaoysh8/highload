Asynchrony in PHP and FPM

If you use PHP-fpm, you have a very convenient option to execute the code asynchronously . The function fastcgi_finish_request () allows you to send a Web server response without stopping the PHP script itself. fastcgi_finish_request

Those. execution of the program does not change in any way, and the function itself resets the output buffer and closes the connection to the Web server. The server in turn sends all this to the client. A convenient opportunity to perform slow operations on the background without significant changes in the program.

Structure

To use the possibility of the background termination of a PHP program, it is necessary that it has the following execution structure:

The main part of the code that is required to send a response to the user.
Call fastcgi_finish_request () .
The heavy part that will be performed on the background and does not contain any logic that affects the response.
Sometimes, if the user needs to show the result of some very slow operation, it is better to divide it into parts. For example, when uploading and converting a video clip:

Return the response immediately after uploading "Thank you, the video is being processed."
Run on the background processing and update the status of this clip in the database.
Example

Sending mail can often take a few seconds. In the classical case, the user will have to wait this time until he sees the answer:

<?
$to = $_POST['email'];

if ( $to )
{
	mail($to, 'Подтверждение', 'Это Ваш ящик?');
	echo 'Подтверждение было отправлено на почту';
}
else
{
	echo 'Вы не ввели все необходимые данные';
}
# If a person enters an email, he will have to wait until the email is sent

Using fastcgi_finish_request (), you can easily turn this code into a quick one:

<?
$to = $_POST['email'];

if ( $to )
{
	echo 'Подтверждение было отправлено на почту';
	fastcgi_finish_request();
	mail($to, 'Подтверждение', 'Это Ваш ящик?');
}
else
{
	echo 'Вы не ввели все необходимые данные';
}
# After calling fastcgi_finish_request (), the visitor will immediately see the response from the server

Sessions

If you use sessions, you must close them before using this function:

<?
...
session_write_close();
fastcgi_finish_request();
The most important

A simple mechanism of asynchronous work with fastcgi_finish_request () will help significantly speed up the response of the site without the need to optimize it.