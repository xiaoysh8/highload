How Blockchain Works

Blockchain is a mechanism for storing and modifying data without central nodes. What is special about it, and why it is necessary to invent some new mechanism, if you can store everything on the server?

The central node problem

Blocking was first described and applied for the provision of digital currency transactions. Consider how the exchange of money occurs usually. Let there are 3 dudes (A, B and C), who have 10 bucks on their account. Their accounts are stored in a bank (in a regular table). When the dude "A" wants to transfer 5 bucks to the dude "B", he will send the request to the bank. The bank will check the state of its balance sheet and send money to the addressee, if they are sufficient on the account. 

There are several problems with this mechanism:

If the bank does not work, customers can not use their accounts and conduct transactions.
The database is somehow accessible to some employees of the bank, who can make changes to it (by mistake or intentionally).
Solving these problems will involve solving two problems:

The list of transactions should not be stored in one single place, but in multiple places at once. So, someone alone can not affect the data, having super-access.
The list of transactions must be public, and everyone can access it. Those. everyone will be able to verify the validity of any transaction.
These principles are the basis of the database block.

Distributed storage

Imagine that three of our guys will keep a table of accounts - each at home. Then, to transfer money from "A" to "B", it will only be necessary to publish an event for the entire network, in which it will be indicated that "A" transfers $ 5 to "B". 

Each node will have the opportunity to make sure that "A" has enough money in the account for the transfer. Well, update the data in your own table. So, even if the node "A" wants to transfer more money than it has, the remaining nodes will not accept this transaction . Openness and the ability to verify any transaction make this system invulnerable to deception.

Transaction Block Chain

In order for any transaction to be verified at any time, the store contains a list of transactions, not a list of accounts:

A -> B: $ 5
B -> B: $ 2
B -> A: $ 1
B -> B: $ 1
...
# Money information in the system is stored as a list of transactions

For convenience, transactions are grouped into blocks, which after assembly are sent throughout the network:

Block 1: A -> B: $ 5
      : B -> B: $ 2
      : B -> A: $ 1

Block 2 : B -> B: $ 1
...
# Block 2 will be filled and sent to all customers

Each client receives such a block and adds it to the already saved blocks. So it turns out a chain of blocks or block (Blockchain - chain of blocks). However, how to make sure that the client received a block with real data, and not forged by other participants?

Confirmation of data

In fact, when one customer sends money to another, his transaction is marked as unconfirmed . Within a short time, it remains so until the special nodes check it.

Such nodes are called miners . To confirm the transaction, they use complex calculations (open to all participants). The winner is the one who will do it before everyone else. In practice, the winner is the one who has more computing power. The winning node receives a small reward, and a new block is added to the chain of the entire system in chronological order. 

After that, the remaining nodes can easily test the validity of the new block with a special hash () function. This ensures synchronization of data among all nodes of the system.

The most important

The block database provides (truly) distributed storage of data. In addition, the openness of the network, allows you to change data (in the example - to transfer money) without using central nodes. That, in turn, keeps the data from fraud and hacking.