Some notes on Python language. Python has been used my main language for the interviews. If I had to use complex data structures like Binary Search Tree, I would go with Java (or C++). A usual misconception about interviews is you need to code in Java or C++. This is WRONG. You should go with the language you're the most confident with. Big companies won't hire you as a language specific engineer, but as a problem solver person.

Because my main language was Python, I found it practical to use during interview. As it's very close to pseudo code, it is very easy to transfer a flow of ideas, into a code.

### What is a variable?
* a container of values
* in c, the variable is a fixed size bucket, that mimght overflow if we add to the bucket to large value than it can contains
* python as names, not variables: names are used as a reference to objects, each referenec has a size of 4bytes on 32 bits machines, otherwise 8 bytes on 64 bits machine

```
x = 300  # 300 is the object, x is the reference (the name pointing to 300). Python increase the ref count of 300
y = [300, 300],  # now 300 as 3 references attached to it. Which means, only one object exists, 

del x # remove reference from 300 (still do), it does not delete objects, but removes that name as a reference to that object

y = None # decrease the ref count to zero
# so now the object can be safely removed
# ref count also goes down when exiting scope
```

If you put large object in the global namespace you might never decrease the ref count to this object

Every object in Python holds three things:
* value
* type
* ref count

The "object is something" tests if x is in the same location as y. 

The Garbage Collector of Python uses 
* Reference counting which is very easy to implement, but is not thread safe, and doesn't  catch cyclical references
* Generational

You can force a class to only contain a set of attributes with:
```
class Point:
	__slots__ = ('x', 'y') # immutable

p = Point()
p.x = 3 # OK
p.name # raise AttributeError 
```

GIL  (Global Interpreter Lock)  prevents Python to run in multiple threads the same code. There is one GIL for each interpreter, so as to prevent reference count to being changed concurrently

To take advantage of multiple CPU, you need to do multi-processing instead of multi threading. Each process will have its own GIL, and it is the responsability of the developper to share information between processes.

Premature optimisation is the root of all evil. Make sure you get the functionality right first. Then ask the question, "Do you even care that it's slow?" Only if the answer is yes do we proceed.


Duck typing is when an operation can be called without needing the type of the object, like "+" for str or int

In Python, favor object composition over class inheritance. Inherit only when it's really convenient. 

| data structure     | search    | insertion | delete    | traverse in order | python object | properties                          |
| ------------------ | --------- | --------- | --------- | ----------------- | :-----------: | ----------------------------------- |
| heap               | O(n)      | O(log(n)) | O(log(n)) | O(nlog(n))        |     heapq     |                                     |
| binary search      | O(logn)   | O(n)      | O(n)      | O(n)              |    bisect     | where to insert in O(log(n))        |
| deque              | O(n)      | O(1)      | O(n)      | O(nlog(n))        |     deque     | implemented as a double linked list |
| stack              | O(n)      | O(1)      | O(n)      | O(nlog(n))        |     list      |                                     |
| binary tree search | O(log(n)) | O(log(n)) | O(log(n)) | O(n)              |       ?       |                                     |
| set                | O(1)      | O(1)      | O(1)      | O(n)              |      set      | implemented as a list               |
| dict               | O(1)      | O(1)      | O(1)      | O(nlogn)          |               | traverse after sorting              |


### How to sort
```sorted(arr, reverse=True, key=lambda x: x[0])``` # x element in arr. To sort dict, you need to make them become a list or an OrderedDict

#### Search where to insert into a sorted array in O(log(n)), Insert in O(n)

```
import bisect

list = [1, 2]
bisect.insort(list, 1.5) => list = [1, 1.5, 2]
bisect.bisect(list, 0) => 0
bisect.insort(list, 3) => list = [1, 1.5, 2, 3]
bisect.bisect(list, 2.5) => 4
```

# Insert in O(log(n)), Traverse in O(nlog(n)), Find the smallest, Find the largest in O(1)
```
from heapq import heappush, heappop

l = []
heappush(l, 2)
heappush(l, 32)
heappush(l, 1)

print(l) => [1, 2, 32]  <- like a heap, not a bst

heappop(l) => 1,list is now [2, 32]

heapq.merge(l1, l2, l3)=> concatenate multiple sorted list

heapq.nlargest(1, l) => first smallest value

l = [1, 3, 2, 4, 5, 1, 4]
heapq.heapify(l) => transform a list into a heap in O(len(list))
print(l) => now it is a heap [1, 3, 1, 4, 5, 2, 4]
```

# Deque, Stack and Queue
append at both end:
```
from collections import deque

deque.append('x') # append to the right
deque.appendleft('y') # append to the left
deque.count(x) # count num value == 
xdeque.pop() # remove and return the right side element
deque.push() # remove and return the left side element
```

Implement stack as list, but NOT queue as stack (not optimized for it!, instead use deque)

# Priority queue
You can use a heapq for this, but there exists a PriorityQueue that you can use
```
from queue import PriorityQueue

q = PriorityQueue()
q.put(b)
q.put(a)

while not q.empty():
	next_item = q.get() # do pop 
```

# Assert and Raises
* You don't continue after an assert. Make sure that you're not in an incorrect state, that the model doesn't know
* Raise, you've messed up, and it is better to raise this issue, and see if we can recover from it
