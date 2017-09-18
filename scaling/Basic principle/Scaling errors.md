Scaling errors

Most of these things may seem innocent at first, but if neglected, they will jeopardize the growth of projects.

Golden Hammer

The concept of a golden hammer comes from an old American saying: if all you have is a hammer, then everything else looks like nails to you. Many developers are a victim of this idea, using only one technology. The price of this is the need to develop and maintain infrastructure based on the chosen technology. On the other hand, in the construction of such a system there can be no need, you choose another technology that is better suited for solving one particular task, and does not need additional efforts to adjust to your needs. The use of technology to solve problems for which it is clearly not intended, is often extremely inefficient .

For example, the general solution to the task of storing key-value pairs is using the DBMS. This choice is usually made because the organization or developers have a good knowledge of DBMS technology (in addition, as a rule, it is already used in the system for other purposes). Problems arise when the database device (usually relational) becomes a bottleneck and prevents scaling . This is because the cost of growth of a solution based on a database is usually significantly more expensive than other available solutions.

There are many alternatives to traditional relational databases for solving this problem.

A big clod of dirt

Dependencies are a necessary evil in most systems and inability to manage dependencies can adversely affect the efficiency.

There are several approaches to managing code dependencies:

Compile all code into one package
Select components of only known versions
All models and services must be backward-compatible
In a large coma of mud, the entire system is an atomic unit. In principle, this has an obvious advantage - the transfer of dependency problems to the compiler. But, at the same time, it connects the solution to the problems of scaling with the deployment of the entire system ... every time! In a similar system, it becomes much more difficult to make any changes.
In the second approach, dependencies are chosen at will, but each component reflects the problems of the first approach at a lower level
The third approach provides clients with backward-compatible interfaces. This allows for a gradual and independent update of all customers. In addition, changes in data occur on the server side, which helps to isolate the client. This approach reduces the problem of dependencies to a minimum
Hero

One of the popular solutions for ensuring the operability of the system is the presence of a "hero" in the team, which takes on the solution of all problems. This approach can be valid in small systems and small teams, when there is one person responsible for the entire technical component. For large systems, with many components, this approach is not scalable (the only person has a limited bandwidth), besides it is risky ... Although they are used very often, the result is inefficiently spent time, which means money.

The best solution to the problem of the Hero is the automation and description of the system. You must have a hero - but they can not be a person, they need to have an automated subsystem.

Not Automation

This problem is strongly related to the previous one and is quite simple.

Without wasting time on automating repetitive tasks, you spend money on those who spend this very time in the future. Automate - it saves your time in the future. Sometimes automation comes to the point of absurdity - do not forget that automation is an intellectual process! Automate the actions that you want to perform once or twice in the history of the application, it's not worth it. this may be more expensive for you than human time.

Among other things, for a man characterized by errors - from this can not escape. In the case of automation, you reduce the risk of the human factor, and again save money, avoiding unnecessary problems.

Monitoring

Monitoring, like testing, is often sacrificed among the first, when the time frame is very stringent (which is now the norm). But how can you solve the problems of a system whose characteristics are not known to you. In large systems, one can not rely on intuition - risks are too great and time is too important. You can fix only the problems that really exist. And only detailed monitoring of the system can tell you about it.