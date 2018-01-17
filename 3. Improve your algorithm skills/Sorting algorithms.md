# Sorting

Sorting a list of items is a major topic that I can't even begin to deal with here (thick books have been written on the subject). I'm just going to mention a few sorting algorithms and their characteristics.

In terms of performance, sorts mostly come in three flavours:

    O(n2)
    O(n.log n) on average, but O(n2) at worst
    O(n.log n)

The first class shouldn't be used for sorting really big arrays, say bigger than 2000 (there is no real limit; it depends on your time constraints). However they are usually very quick to code and debug, so always consider using them unless you are sure that they won't cut it. The algorithms in the second class are very seldom O(n2), so it is fine to use them in most applications, particularly in computer olympiads where you only have 5 or 10 test cases and are highly unlikely to get a really slow case, and even if you do you only lose some of your points. These algorithms can actually be faster than those in the third class because they are simpler.

Each of the sorts below has a description of the algorithm, the efficiency, pros and cons and other notes.
### Bubble sort

Algorithm
    Scan through the array from left to right, comparing adjacent elements. Whenever two adjacent elements are out of order, swap them. Repeat this until the array is sorted. 
Efficiency
    O(n2) 
Pros

        Simple to implement and understand, but not much more so than several more efficient sorts
        Will be very quick on an already sorted list

Cons

        Probably the slowest sort ever invented

Notes
    Don't bother learning this sort. There are other sorts which are just as easy to implement and much faster. Although there are some time saving modifications that can be made (e.g. keeping track of the last swap in the previous pass, and only going up to there in the current pass), they aren't worth it. 

### Selection sort

Algorithm
    Search the array for the largest element, and swap it with the element at the end. Then repeat for just the first n - 1 elements, then for the first n - 2 etc. This extracts the elements individually in reverse order. 
Efficiency
    O(n2) 
Pros

        Easy to implement

Cons

        Is no faster on a partially sorted array

Notes
    This is a reasonable sort to use for sorting a fairly small number of items, and my personal favourite of the O(n2) sorts. 

### Insertion sort

Algorithm
    The first part of the list is sorted and the remainder is unsorted. To begin with, all elements are in the unsorted part. Go through the elements in the unsorted part sequencially and insert each into the sorted part by searching for it's correct position with a linear search or a binary search. 
Efficiency
    O(n2) 
Pros

        Fairly easy to implement with a linear search
        If a backwards linear search is used, it is much faster on an partially sorted array

Cons

        On an array, insertion is slow because all the elements after it have to be moved up.

Notes
    This sort isn't especially useful by itself (although it is a respectable O(n2) sort), but forms the basis of Shell sort. 

### Shell sort

Algorithm
    Consider the sequence 1, 4, 13, 40, 121 etc, where each term is triple the previous one plus one. Go through the sequence backwards, starting with the largest term less than the number of items being sorted. For each member of the sequence g, do the following

        Sort the elements with indices 1, 1 + g, 1 + 2g, 1 + 3g ...
        Sort the elements with indices 2, 2 + g, 2 + 2g, 2 + 3g ...
        Sort the elements with indices 3, 3 + g, 3 + 2g, 3 + 3g ...
        And so on, up to g, 2g, 3g, 4g, ...

    Each of these sorts should be done with a sort that works quickly on a partially sorted list; a good choice is Insertion sort. 
Efficiency
    At worst O(n3/2). It's also possible to use sequences other than 1, 4, 13, ... but they do not necessarily give the same efficiency. 
Pros

        Usually much faster than regular insertion sort for very little extra work
        The fastest of the non-recursive sorts shown here
        Easy to get working

Cons

        Running time is slighly sensitive to the data (but far less so than binary tree sort or Quicksort)
        Not quite as fast as ultra-fast sorts like Quicksort
        If you mess up the sequence, you get a sort that will still work but is slow.

Notes
    This is an excellent medium/heavy sort, and after trying this out I decided to use this whenever I need a large number of items sorted in a hurry and I don't mind it being a couple of times slower than a heavy duty sort. 

### Quicksort

Algorithm
    Save the value of the middle element of the array. Have a "left" pointer that starts at the first element and moves right and a "right" pointer that starts at the last element and moves left. Repeat the following operation until the pointers cross:

        Scan with the left pointer until an element larger or equal to the saved value is found.
        Scan with the right pointer until an element smaller or equal to the saved value is found.
        If the pointers haven't crossed yet, swap the values they point to and manually advance both pointers one position.

    Then recursively sort the elements in the ranges [start, right] and [left, end]. 
Efficiency
    O(n.log n) on average 
Pros

        Extremely fast on average

Cons

        Fairly tricky to implement and debug
        Very slow in the worst case (but see notes)
        In the worst case, could cause a stack overflow

Notes

    If the middle element in the array is very large or very small, the sort becomes slower. To reduce the chances of this, you can take median of the first, middle and last elements and put it in the middle.

    In a degenerate case, Quick sort could also recurse to a very deep level and cause a stack overflow. One way to avoid this is to keep track of the level of recursion (e.g. by passing it as a parameter) and switch to a simpler sort (like selection sort) if the recursion is too deep.

### Heap sort

Algorithm
    Add the elements to a heap one at a time. Then remove the elements from the top of the heap one at a time, and they will be in order. If the heap is done upside down (i.e. largest element at the root) then the incomplete sorted list and the heap can share the same array without requiring extra memory. 
Efficiency
    O(n.log n) 
Pros

        Guaranteed to be O(n.log n)
        One of the few non-recursive O(n.log n) sorts
        Fairly easy to debug, because it is non-recursive and because it consists of two smaller and unrelated parts

Cons

        Not as fast on average as Quick sort
        Quite a lot of code to implement the heap operations

Notes

    This isn't an especially useful sort, but the running time is guaranteed O(n.log n) and it is very light on memory consumption. Also, if you have already implemented a heap for some other reason, then this sort comes almost for free.

    It is possible to get some extra speed by constructing the heap in one pass: put everything into the heap ignoring the heap condition, then work through the heap from bottom to top, bubbling elements down the heap where necessary. This makes the heap creation take linear time.

### Merge sort

Algorithm
    Split the array in half and recursively sort each half. Then merge the two halves by keeping a pointer to the front of each list and continuously taking the smaller of the two elements. 
Efficiency
    O(n.log n) 
Pros

        Guaranteed to be O(n.log n)
        Simple to understand

Cons

        Not as fast as Quick sort on average
        Requires as much memory as the original array

Notes

    It is convenient to sort the right half backwards, and have the pointer to the second half move backwards. Then you never have to check whether either list is empty; the pointer simply moves to the biggest element in the other and is never used again.

    I used to like this sort because it is guaranteed O(n.log n), but it is a memory hog and not as fast as Quicksort in practice.

### Binary tree sort

Algorithm
    Insert each element sequencially into a binary search tree. The do an in-order walk of the tree (if you don't know what this is, read the information on binary search trees). 
Efficiency
    O(n.log n) on average 
Pros

        If you've already implemented binary search trees, then this sort is quite easy.

Cons

        Hogs memory for the pointers
        Not very pleasant to implement or debug
        Not very fast
        Is O(n2) on a sorted list

Notes
    Think of this as the bubble sort of the medium speed sorts: slow and not very useful. Binary search trees are more useful if you are mixing things like insertions, searches, and extraction of sorted data. 

