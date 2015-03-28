A C++ implementation of an LRU cache that has a maximum size (but no expiration time).  The cache is implemented as a single hash table and provides fast, constant-time insertion, retrieval and query operations.

# Usage #

```
#include <iostream>
#include "lru.hpp"

using namespace std;

int compute_value(int key)
{
    return 100 + key;
}

int main()
{
    // Create a cache that can hold 10 (int, int) pairs at most
    typedef plb::LRUCacheH4<int, int> lru_cache;
    lru_cache cache(10);

    // Insert a (key, value) pair into the cache: same syntax as std::map
    cache[1] = compute_value(1);
    
    for (int key = 1;  key <= 2;  ++key) {
        // Query the cache: Does it contain the key?
        lru_cache::const_iterator it = cache.find(key);

        if (it != cache.end()) {
            // Key found: retrieve its associated value
            cout << "retrieving: " << it.key() << " -> " << it.value() << endl;
        }
        else {
            // Key not found: compute and insert the value
            cache[key] = compute_value(key);
            cout << "inserting: " << key << " -> " << cache[key] << endl;
        }
    }

    return 0;
}
```

# Design #

  * The cache is implemented as a single hash table.  Most implementations I have seen use a map and a list. They have the same algorithmic complexity (constant time insertion, retrieval and query) but the few experiments I have run show that this implementation is about 5 times faster (see lru\_comp.cpp).

  * A bucket is a (key, value, older`*`, newer`*`) tuple, where older and newer are pointers to buckets.  The MRU has a NULL newer pointer, and the LRU has a NULL older pointer.

  * The LRU elment is evicted when inserting a new key in a full cache; the operation is constant-time.

  * The cache provides methods similar to those of the STL containers; especially it provides acces to its elements through constant iterators:

```
template<class K, class V>
class LRUCacheH4
{
    V & operator[](const K & key);
	
    int size() const;
    int maxsize() const;
    bool empty() const;
	
    const_iterator find(const K & key);         // updates the MRU
    const_iterator find(const K & key) const;   // does not update the MRU
    const_iterator mru_begin() const;           // from MRU to LRU
    const_iterator lru_begin() const;           // from LRU to MRU
    const_iterator end() const;

    ...
};

```

## Limitations ##

  * The cache is not safe against concurrent modifications (i.e. not thread-safe).

  * The cache allocates all the memory it may need right in the constructor.  This makes the implementation easier (we don't have to deal with reallocation and invalid pointers) at the expense of a high up-front memory footprint.  This could prove too costly for applications that create many caches but fill only some of them.

  * The implementation depends on hashtable.h from GNU;  that header may not be available on your platform.

## Performance ##

The directory test/ from the source distribution contains two kinds of tests: performance tests (CPU and memory), and correctness tests.

The performance tests can be run on this implementation only, or they can also be run on another implementation to provide comparisons between a hash table design to a map + list design.

The correctness tests are made of unit tests (`lru_tests.cpp`) and integrated tests, which compare the contents of this cache to the contents of another implementation.

### Files ###

  * `smaps.hpp`, `smaps.cpp`, `smaps_test.cpp`, `smaps.txt`: An interface to Linux /proc/nnn/smaps.  Used to get the memory usage.

  * `lru_tests.cpp`: unit tests.

  * `lru_comp.cpp`: performance tests, and integration tests.  You can pass two arguments when running `lru_comp`:
    * a test case which is `TEST_CASE_INSERT` or `TEST_CASE_INSERT_READ`
    * an action which is `RUN_PLB`, `RUN_PA` or `CORRECTNESS`

# References #

## Implementation ##

I have used the following documentation when implementing the cache:

  * http://gcc.gnu.org/onlinedocs/libstdc++/libstdc++-html-USERS-4.2/hashtable_8h-source.html

## Other Implementations ##

  * The closest implementation I have found is the one from Patrick Audley: http://patrickaudley.com/code/project/lrucache.  Its interface is similar (although the signatures are a bit different).  It provides synchronisation and it does not allocate all the memory up-front.  However, it uses the map + list implementation, which is slower and requires more memory when the cache is full.  This is the implementation I have used for performance comparisons and for integration tests.


## Misc ##

The following are either implementations, "napkin designs" or discussions of LRU caches in C++:

  * http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.88.7296&rep=rep1&type=pdf
  * http://www.bottlenose.demon.co.uk/article/lru.htm
  * http://stackoverflow.com/questions/2057424/lru-implementation-in-production-code
  * http://stackoverflow.com/questions/2504178/lru-cache-design
  * http://www.megasolutions.net/cplus/STL-based-LRU-cache,-any-suggestions-for-improvements_-60106.aspx
  * http://www.codeproject.com/KB/recipes/LRUCache.aspx
  * http://bytes.com/topic/c/answers/653919-design-patterns-lru-cache-c