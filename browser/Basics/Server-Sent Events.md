Server-Sent Events

SSE is the technology for sending server notifications to the browser. The client as it is connected to a stream of updates and automatically receives notifications in case of new events. Server-Sent Events example

Polling the server (polling and long polling)

The idea is as follows: The AJAX application checks the new data on the server. The client creates a request and waits for a response. The server returns a response, even if there are no updates. So the process creates an additional load on the channel.

Therefore, "hanging" requests are used in addition - if there are no updates, the server supports an open connection. When new data appears, the server sends them and closes the connection. After that, the process is repeated.

WebSocket or SSE

The Server-Sent Events technology opens a one-way connection between the server and the client, after which the server sends data to the application when it is required (push notifications). Another difference is that SSEs are handled directly on the client side, which only listens to the channel.

WebSocket is another messaging technology between the client and the server. Its main difference is that the communication is two-way, the client receives and sends messages. WebSocket scheme

It is suitable for advanced web applications, games, instant messengers, etc., where real-time communication is required.

Server-Sent Events are transmitted over HTTP, no special protocol or separate server is required. While the WebSocket requires full duplex communication and additional servers with the protocol. In addition, SSE supports reconnection, event IDs and arbitrary events.

Implementation on the client side

The API Server-Sent Events is built on the EventSource interface . Therefore, to open a new connection with the server, you need to create an EventSource object. It specifies the path to the script that creates the events:

var evtSource = new EventSource("ssedemo.php");

# если скрипт лежит на другом домене, то так:

var evtSource = new EventSource("//api.example.com/ssedemo.php", { withCredentials: true } );
# SSE on the client side is implemented in JavaScript, the server script is in PHP, ASP, NodeJS or other convenient language

After the connection is initialized, the client can receive messages:

evtSource.onmessage = function(e) {
  var newElement = document.createElement("li");
  
  newElement.innerHTML = "message: " + e.data;
  eventList.appendChild(newElement);
}
# Receives messages from the server and adds the message text to the HTML list

And by using the addEventListener () directive, you can also receive events:

evtSource.addEventListener("ping", function(e) {
  var newElement = document.createElement("li");
  
  var obj = JSON.parse(e.data);
  newElement.innerHTML = "ping at " + obj.time;
  eventList.appendChild(newElement);
}, false);
# Called automatically when the server sends a ping event, the JSON client parses and outputs the result

In addition to its own events, standard events are also used:

onopen - if the connection is successfully opened;
onerror - in case of connection failure;
onmessage - a new message.
The simplest example of standard events would look like this:

var eventSource = new EventSource('sse_demo');

eventSource.onopen = function(e) {
  console.log("Открыто соединение");
};

eventSource.onerror = function(e) {
  if (this.readyState == EventSource.CONNECTING) {
    console.log("Ошибка соединения, переподключение");
  } else {
    console.log("Состояние ошибки: " + this.readyState);
  }
};

eventSource.onmessage = function(e) {
  console.log("Сообщение: " + e.data);
};
# Attempted reconnection in case of an error

Please note that there may be several events.

Server-side implementation

The script that sends events must use the MIME type text / event-stream . Each message is a simple text separated by two characters of a new line (\ n \ n) (an empty string).

Our backend in PHP will look like this:


date_default_timezone_set("America/New_York");
header("Content-Type: text/event-stream\n\n");

$counter = rand(1, 10);
while (1) {
  # Отправлять событие "ping" каждую секунду
  
  echo "event: ping\n";
  $curDate = date(DATE_ISO8601);
  echo 'data: {"time": "' . $curDate . '"}';
  echo "\n\n";
  
   $counter--;
  
  if (!$counter) {
    echo 'data: Время сообщения ' . $curDate . "\n\n";
    $counter = rand(1, 10);
  }
  
  ob_end_flush();
  flush();
  sleep(1);
}
# Randomly displays a message with a timestamp

The most important thing

Use Server-Sent Events if you need a one-way communication between the server and the client - output notifications, updates. For more complex tasks, WebSocket is suitable.