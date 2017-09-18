What is a hash table and how does it work?

A hash table is a special data structure for storing key pairs and their values. In essence, this is an associative array, in which the key is represented as a hash function.Hash table

Perhaps the main property of hash tables is that all three operations are inserted, search and delete are performed on average in O (1) time, the average search time for it is also O (1) and O (n) in the worst case.

Simple representation of hash tables

To understand what a hash table is, imagine that you were asked to create a library and fill it with books. But you do not want to fill cabinets in random order.

The first thing that comes to mind is to put all the books in alphabetical order and write everything in a certain directory. In this case, you do not have to search the necessary book for the entire library, but only for the reference book.

And you can make it even more convenient. If you initially start from the title of the book or the name of the author, it is better to use a hash algorithm that processes the input value and gives the number of the cabinet and shelves for the desired book.

Knowing this hashing algorithm, you will quickly find the right book by its name.

Note that the hash function must have the following properties:

Always return the same address for the same key;
It does not necessarily return different addresses for different keys;
Uses the entire address space with the same probability;
Quickly calculate the address.
The fight against collisions (they are the same collision)

In the ideal case, when all key-value pairs are known in advance, it is easy to implement an ideal hash table in which the search time will be constant (using an ideal hash function that determines the positions in the table by integer values ​​and without collisions).

But in most cases you have to fight with collisions. Usually, the methods of chains and open indexing are used.

Chain method

This method is often called open hashing . Its essence is simple - elements with the same hash fall into one cell in the form of a linked list . Hash chaining

That is, if the cell with the hash is already occupied, but the new key differs from the already existing one, the new element is inserted into the list in the form of a key-value pair.

If the chain method is selected, the insertion of the new element occurs after O (1), and the search time depends on the length of the list and in the worst case is equal to O (n). If the number of keys n , a is distributed over m- cells, then the ratio n / m is the filling factor.

In C ++, the chaining method is implemented as follows:

class LinkedHashEntry {
private:
      int key;
      int value;
      LinkedHashEntry *next;
public:
      LinkedHashEntry(int key, int value) {
            this->key = key;
            this->value = value;
            this->next = NULL;
      }
 
      int getKey() {
            return key;
      }
 
      int getValue() {
            return value;
      }
 
      void setValue(int value) {
            this->value = value;
      }
 
      LinkedHashEntry *getNext() {
            return next;
      }
 
      void setNext(LinkedHashEntry *next) {
            this->next = next;
      }
};
# Check the cell and create a list

Open indexing (or closed hashing)

The second common method is open indexing. This means that key-value pairs are stored directly in the hash table. And the insert algorithm checks the cells in some order, until an empty cell is found. The order is calculated on the fly. Open addressing linear probing

The simplest sequence of samples is linear sampling (or linear analysis). Here everything is simple - in case of a collision, the following cells are checked linearly until an empty cell is found.

A search algorithm searches for cells in the same order as when inserted, until they find the desired item or an empty cell that indicates that the key is missing. In case the table is filled, it will have to be dynamically expanded.

The method of linear probing for open indexing on C ++:

class HashEntry {
private:
      int key;
      int value;
public:
      HashEntry(int key, int value) {
            this->key = key;
            this->value = value;
      }
 
      int getKey() {
            return key;
      }
 
      int getValue() {
            return value;
      }
 
      void setValue(int value) {
            this->value = value;
      }
};
# Check cells and insert values

The most important thing

Hashing and hash tables are used for more convenient storage of key-value pairs. If you need maximum efficiency, then using a hash table with lists will be much faster than a regular table.