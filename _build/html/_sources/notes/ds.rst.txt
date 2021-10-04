.. include:: <isopub.txt>
.. include:: <mmlalias.txt>

PHP Data Structures Extension
=============================

Introduction
------------

The `PHP 7 Data Structures Extension <https://www.php.net/manual/en/book.ds.php>`_  provides classical data structures for use in PHP. It is more flexible and efficient than PHP's one-fize-fits-all ``array``.
`Efficient Data Structures for PHP <https://medium.com/@rtheunissen/efficient-data-structures-for-php-7-9dda7af674cd>`_ explains its classes *Vector*, *Deque*, *Map*, *Set*, *Stack*, *Queue*, *PriorityQueue* and *Pair* and their implemented interfaces\ |mdash|\ *Collection*, *Hashable* and *Sequence*.

See also:

* This `Reddit thread <https://www.reddit.com/r/PHP/comments/b6ffs5/who_here_uses_ds_data_structures_and_for_which/>`_.
* The answers to questions at the end of the **Ds** github repository `README <https://github.com/php-ds/ext-ds>`_.

I believe **polyfill** is a `related extension <https://github.com/php-ds/polyfill>`_. 

Important Comment on Installation
---------------------------------

If you are using **nginx**, you must restart the PHP fascgi process after enabling the **Ds** extension: 

.. code-block:: bash

    sudo systemctl restart php7.4-fpm

Summary of Built&ndash;in PHP Interfaces
----------------------------------------

Understanding the built&ndahs;in PHP interfaces helps in understanding the ``\Ds`` interfaces and classes. 

Traversable Interface
~~~~~~~~~~~~~~~~~~~~~

To enable iteration in a ``foreach`` loop of a user&ndash;defined class, you must support the ``\Traversable`` interface, but you cannot implement ``\Traverable`` directly in user&ndash;defined classes directly becasue ``\Traversable`` is a internal PHP
engine interface. Instead you must implement either ``IteratorAggregate`` or ``Iterator``. 

.. note:: Since the ``\Ds`` extension extends the PHP engine itself, the the preceding comments don't apply to it, only to user&ndash; defined classes. 

Iterator Interface
~~~~~~~~~~~~~~~~~~

``Iterator`` is the interface for external iterators or objects that can be iterated themselves internally.

Iterator Interface synopsis 
+++++++++++++++++++++++++++

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

1. Before the first iteration of the loop, Iterator::rewind() is called.
2. Before each iteration of the loop, Iterator::valid() is called.
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

Countable Iterface
~~~~~~~~~~~~~~~~~~

Countable Interface synopsis
++++++++++++++++++++++++++++

.. code-block:: php

    <?php
    interface Countable {
        /* Methods */
        public count(): int
    }

``Countable::count`` returns the number of elements in an object.

JSONSerializable 
~~~~~~~~~~~~~~~~~

Objects implementing ``JsonSerializable`` can customize their JSON representation when encoded with ``json_encode()``.

JSONSerializable Interface synopsis
+++++++++++++++++++++++++++++++++++

JSONSerializable  Interface synopsis
++++++++++++++++++++++++++++++++++++

.. code-block:: php

    <?php
    interface JsonSerializable {
        /* Methods */
        public jsonSerialize(): mixed
    }

Use ``JsonSerializable::jsonSerialize()``  to specify data which should be serialized to JSON.

Ds Interfaces
-------------

``Collection`` interface synopsis:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Collection interface covers functionality common to all the data structures in this library. It guarantees that all structures are *traversable*, *countable*, and can be converted to json using *json\_encode()*, and it thereby  
provides support for *foreach*, *echo*, *count*, *print\_r*, *var\_dump*, *serialize*, *json\_encode*, and *clone*.

.. code-block:: php

    class Ds\Collection implements Traversable, Countable, JsonSerializable {
        /* Methods */
        abstract public clear(): void
        abstract public copy(): Ds\Collection
        abstract public isEmpty(): bool
        abstract public toArray(): array
    }

The PHP **clone** reserved word invokes an object's "copy constructor", which can be implemented through a custom ``__clone() : void``. See PHP `Object Cloing <https://www.php.net/manual/en/language.oop5.cloning.php>`_.

Method Descriptions
~~~~~~~~~~~~~~~~~~~
    
    Ds\Collection::clear — Removes all values
    Ds\Collection::copy — Returns a shallow copy of the collection
    Ds\Collection::isEmpty — Returns whether the collection is empty
    Ds\Collection::toArray — Converts the collection to an array

Example Code
~~~~~~~~~~~~

.. code-block:: php

    <?php
    $collection_a = new \Ds\Vector([1, 2, 3]);
    $collection_b = new \Ds\Vector();
    
      var_dump($collection_a, $collection_b);
       /*
           object(Ds\Vector)[1]
           public 0 => int 1
           public 1 => int 2
           public 2 => int 3
    
           object(Ds\Vector)[2]
       */
    
      //json_encode
      var_dump( json_encode($collection_a));
       /*
       string '[1,2,3]
       */
     
       //count
      var_dump(count($collection_a));
       /*
       int 3
       */
     
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
      //浅拷贝， shallow copy
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
