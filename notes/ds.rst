.. include:: <isopub.txt>
.. include:: <mmlalias.txt>

PHP Data Structures Extension
=============================

Introduction
------------

The `Data Structures Extension <https://www.php.net/manual/en/book.ds.php>`_ for PHP 7 provides classical data structures for use in PHP. The contains the container classes  *Vector*, *Deque*, *Map*, *Set*, *Stack*, *Queue*, *PriorityQueue* and *Pair* 

Create class and interface grarphic--inheritance and implements.

    class Ds\Vector implements Ds\Sequence, ArrayAccess { ...} 
    class Ds\Deque implements Ds\Sequence, ArrayAccess {
    class Ds\Map implements Ds\Collection, ArrayAccess {
    class Ds\Pair implements JsonSerializable {
    class Ds\Set implements Ds\Collection, ArrayAccess {
    class Ds\Stack implements Ds\Collection, ArrayAccess {
       class Ds\Queue implements Ds\Collection, ArrayAccess {

    
 It is more flexible and efficient than PHP's one-fize-fits-all ``array``.
`Efficient Data Structures for PHP <https://medium.com/@rtheunissen/efficient-data-structures-for-php-7-9dda7af674cd>`_ explains its classes *Vector*, *Deque*, *Map*, *Set*, *Stack*, *Queue*, *PriorityQueue* and *Pair* and their implemented interfaces\ |mdash|\ *Collection*, *Hashable* and *Sequence*.

See also:

* This `Reddit thread <https://www.reddit.com/r/PHP/comments/b6ffs5/who_here_uses_ds_data_structures_and_for_which/>`_.
* The answers to questions at the end of the **Ds** github repository `README <https://github.com/php-ds/ext-ds>`_.

The `polyfill <https://github.com/php-ds/polyfill>`_ repository provides IDE integration. It can also be installed using Composer.

Important Comment on Installation
---------------------------------

If you are using **nginx**, you must restart the PHP fascgi process after enabling the **Ds** extension: 

.. code-block:: bash

    $ sudo systemctl restart php7.4-fpm

Summary of Built\ |ndash|\ in PHP Interfaces
--------------------------------------------

Understanding the built\ |ndash|\ in PHP interfaces helps in understanding the ``\Ds`` interfaces and classes. 

Traversable Interface Synopsis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To iterate a  a user\ |ndash|\ defined class with ``foreach``, you must support the ``\Traversable`` interface, but since you cannot implement ``\Traverable`` directly in user\ |ndash|\ defined classes becasue ``\Traversable`` is a internal PHP
engine interface, you must instead implement either ``IteratorAggregate`` or ``Iterator``. 

.. note:: The Data Structures extension extends the PHP engine itself, so the preceding comments don't apply to it. They only apply to user\ |ndash|\ defined classes.
 
Iterator Interface Synopsis 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Iterator`` is the interface for external iterators or objects that can be iterated themselves internally.

.. code-block:: php
    
    <?php
    interface Iterator extends Traversable {
        /* Methods */
        public current(): mixed  // Returns current element
        public key(): mixed
        public next(): void
        public rewind(): void
        public valid(): bool
    }

Examples of ``Iterator`` uses: 

``SplFileObject`` implments the ``Iterator`` and can be usedin a ``foreach`` loop to iteratre over the lines of a file or rows of a CSV file. Other built-in PHP classes like ``FilterIterator``, ``RegexIterator`` and so on  also implement ``Iterator``, often by implementing a
an interface derived from ``Iterator``.

Explanation
+++++++++++

The methods should be implemented so ``current`` returns the current element. ``current()`` is the PHP analogue of the dereference opertor of C++ stl-compliant iterator classes. ``key()`` returns the key of the current element. ``next()`` is analogous to a C++ iterator's 
``iterator& operator++()``.  ``rewind()`` resets the iterator to the first element in the collection, and ``Iterator::valid(): bool``, which is called immediately after ``Iterator::rewind()`` and ``Iterator::next()``, check if the current position is valid, i.e., whether 
a ``foreach`` loop should end because is "past" the last element.

A ``foreach`` loop invokes ``Iterator`` methods in this order:

1. Before the first iteration of the loop, ``Iterator::rewind()`` is called.
2. Before each iteration of the loop, ``Iterator::valid()`` is called.
3. If ``Iterator::valid()`` returns false, the loop is terminated; otherwise, it continues and ``Iterator::valid()`` and ``Iterator::key()`` are called.
4. The loop body is evaluated.
5. After each iteration of the loop, ``Iterator::next()`` is called and we go to step #2.

This is roughly equivalent to:

.. code-block:: php

    <?php
        $it->rewind();
        
        while ($it->valid()) {

            $key = $it->key();
            $value = $it->current();
        
            // ... foreach-body code here
        
            $it->next();
        }

Countable Interface Synopsis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    <?php
    interface Countable {
        /* Methods */
        public count(): int
    }

This interfaces allows an object instance to be use with PHP's ``count($my_object)`` method.  Implement ``Countable::count`` so it returns the number of elements in the object.

JSONSerializable  Interface synopsis
++++++++++++++++++++++++++++++++++++

Objects implementing ``JsonSerializable`` can customize their JSON representation when encoded with ``json_encode()``.

.. code-block:: php

    <?php
    interface JsonSerializable {
        /* Methods */
        public jsonSerialize(): mixed
    }

Use ``JsonSerializable::jsonSerialize()``  to specify data which should be serialized to JSON.

Ds Interfaces
-------------

``Ds\Collection`` interface
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Collection interface covers functionality common to all the data structures in this library. It guarantees that all structures are *traversable*, *countable*, and can be converted to json using *json\_encode()*, and it thereby  
provides support for *foreach*, *echo*, *count*, *print\_r*, *var\_dump*, *serialize*, *json\_encode*, and *clone*.

The PHP **clone** reserved word invokes an object's "copy constructor", which can be implemented through a custom ``__clone() : void``. See PHP `Object Cloing <https://www.php.net/manual/en/language.oop5.cloning.php>`_.

.. code-block:: php

    <?php
    class Ds\Collection implements Traversable, Countable, JsonSerializable {
        /* Methods */
        abstract public clear(): void
        abstract public copy(): Ds\Collection
        abstract public isEmpty(): bool
        abstract public toArray(): array
    }

Method Descriptions
~~~~~~~~~~~~~~~~~~~
    
.. code-block:: php

    <?php
    Ds\Collection::clear() ??? Removes all values
    Ds\Collection::copy() ??? Returns a shallow copy of the collection
    Ds\Collection::isEmpty() ??? Returns whether the collection is empty
    Ds\Collection::toArray() ??? Converts the collection to an array

Ds\Vector Demonstration of its Ds\Collection interface
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    <?php
    $collection_a = new \Ds\Vector([1, 2, 3]);
    $collection_b = new \Ds\Vector();
    
     var_dump($collection_a, $collection_b);

     //json_encode
     var_dump( json_encode($collection_a));

     //count
      echo count($collection_a);

.. raw:: html

    <pre>
           object(Ds\Vector)[1]
           public 0 => int 1
           public 1 => int 2
           public 2 => int 3
    
           object(Ds\Vector)[2]
    
   
       string '[1,2,3]   //json_encode
    
       3                 // count($collection_a) 
    </pre>

.. code-block:: php

      <?php     
       // serialize
       var_dump(serialize($collection_a));
       /*
       string 'C:9:"Ds\Vector":12:{i:1;i:2;i:3;}'
       */
    
       // foreach
       foreach ($collection_a as $key => $value) {
          echo $key ,'--', $value, PHP_EOL;
       }
       /*
          0--1
          1--2
          2--3
        */
    
       // clone
       $clone = clone($collection_a);
       var_dump($clone);
       /*
         object(Ds\Vector)[1]
         public 0 => int 1
         public 1 => int 2
         public 2 => int 3
       */
    
       // push
       $clone->push('aa');
       var_dump($clone);
       /*
       object(Ds\Vector)[3]
         public 0 => int 1
         public 1 => int 2
         public 2 => int 3
         public 3 => string 'aa' (length=2)
        */
    
      // isEmpty
      var_dump($collection_a->isEmpty(), $collection_b->isEmpty());
       /*
       boolean false
       boolean true
        */
    
      // toArray
      var_dump($collection_a->toArray(), $collection_b->toArray());
       /*
        array (size=3)
         0 => int 1
         1 => int 2
         2 => int 3
    
       array (size=0)
         empty
       */

      // copy ( void )
      //???????????? shallow copy
      $collection_c = $collection_a->copy();
    
      var_dump($collection_c);
       /*
       object(Ds\Vector)[3]
         public 0 => int 1
         public 1 => int 2
         public 2 => int 3
       */
    
      $collection_c->push(4);
      var_dump($collection_a, $collection_c);
       /*
       object(Ds\Vector)[1]
         public 0 => int 1
         public 1 => int 2
         public 2 => int 3
    
       object(Ds\Vector)[3]
         public 0 => int 1
         public 1 => int 2
         public 2 => int 3
         public 3 => int 4
       */
     
       // clear
       $collection_a->clear();
       $collection_b->clear();
       $collection_c->clear();
    
       var_dump($collection_a, $collection_b, $collection_c);
       /*
       object(Ds\Vector)[1]
       object(Ds\Vector)[2]
       object(Ds\Vector)[3]
       */

``Ds\Hashable`` interface
~~~~~~~~~~~~~~~~~~~~~~~~~

Hashable is an interface which __allows objects to be used as keys__. It???s an alternative to ``spl_object_hash()``, which determines an object???s hash based on its handle: this means that two objects that are considered equal by an implicit definition would not treated as equal because they are not the same instance.

``hash($my_object)`` is used to return a scalar hash value to be used as the object's hash value, which determines where it goes in the hash table. While this value does not have to be unique, objects which are equal must have the same hash value.

``$my_object->hash()`` returns a scalar hash key to determines where ``$my_object`` should go in the hash table (--but WHICH hash table? Are certain Ds classes implemented as a has table, and must clients, in order to insert into them, derive user-defined classes from ``Ds\Hashable``?)
While this value does not have to be unique, objects which are equal must have the same hash value.

``$myobject->equals($other_object)`` is used to determine if two objects are equal. It's guaranteed that the comparing object will be an instance of the same class as the argument.

    <?php
    class Ds\Hashable {
        /* Methods */
        abstract public equals(object $obj): bool
        abstract public hash(): mixed
    }


    Ds\Hashable::equals ??? Determines whether an object is equal to the current instance
    Ds\Hashable::hash ??? Returns a scalar value to be used as a hash value
