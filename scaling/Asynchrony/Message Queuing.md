Message Queuing

The message queue allows asynchronous execution of program sections. This allows:

Increase application speed
Serving more visitors (scaled)
Use different development languages ​​in one application
The queue system is a principle, not a specific technology. It is not necessary to use an external solution to implement a queue system. You can implement a queue, say, MySQL and PHP. However, the simplicity and availability of ready-made solutions will make it faster.

Cron

Using cron scripts (for example, for converting video files) is the most primitive method for implementing queues. Modern systems make everything much easier and more convenient.

The queue system consists of two main components.

Queue Server

The queue server stores a list of messages (or tasks, job queue) that the main application sends to it. The task is just information about what and how to perform. Job queue server

The queue server itself does nothing. Its only task is to store the queue itself.

Handler

A handler (or worker) is part of the main program that works with the queue in the opposite direction. It receives new messages from the queue and performs the corresponding actions. Job queue worker

Those. The slow portion of the program is removed from the main program and transferred to the worker. In the main application, it replaces to send the message to the queue.

Integration

The general structure of the message system in applications looks like this: Job queue

For work you will need:

Install a queue system (for example, Gearman ).
Replace the slow section of the program to send the message.
Develop a handler, i.e. to move a slow site there.
Start the vorker in one or more threads.
Install a monitoring system to track queue load and status of vorkers.
Scaling

From the point of view of scaling, the queue system provides great opportunities.

The opportunity to realize firemen on different technologies will allow using the most effective solutions.
Using multiple message servers will make the program reliable and scalable.
The ability to run several vorkers will speed up slow operations.
Many messaging systems support prioritization, which will allow faster execution of important tasks.
Practice and most important

Read about the application of the Gearman queue system in PHP . Queue systems can and should be used not only on large Web sites. Slow operations significantly worsen the experience of using the application, and the queues will provide a high speed for visitors.