# cs-interpreting-performance-profiles-readme


## Overview

This README reviews results from the previous lab an introduces yet another implementation of the `List` interface, the doubly-linked list.


## Objectives

1.  Interpret results from the previous lab.
2.  Understand the implementation of the doubly-linked list.
3.  Analyze the performance of doubly-linked list operations.


## Performance profiling results

In the previous lab, we used `Profiler.java` to run various `ArrayList` and `LinkedList` operations with a range of problem sizes.  We plotted the runtime versus problem size on a log-log scale and estimated the slope of the resulting curve, which indicates the leading exponent of the relationship between run time and problem size.  For example, when we used the `add` method to add elements to the end of an `ArrayList`, we found that the total time to perform `n` adds was proportional to `n`; that is, the estimated slope was close to 1.  We concluded that performing `n` adds is in O(`n`), so on average the time for a single add is constant time, or O(1), which is what we expected based on algorithm analysis.

The lab asked you to fill in the body of `profileArrayListAddBeginning`, which tests the performance of adding new elements at the beginning of an ArrayList.  Based on our analysis, we expect each add to be linear, because it has to shift the other elements to the right; so we expect `n` adds to be quadratic.

Here's our solution to the exercise, which you can see and run by checking out the `solutions` branch of the lab:

```java
	public static void profileArrayListAddBeginning() {
		Timeable timeable = new Timeable() {
			List<String> list;

			public void setup(int n) {
				list = new ArrayList<String>();
			}

			public void timeMe(int n) {
				for (int i=0; i<n; i++) {
					list.add(0, "a string");
				}
			}
		};
		int startN = 4000;
		int endMillis = 10000;
		runProfiler("ArrayList add beginning", timeable, startN, endMillis);
	}
```

This method is almost identical to `profileArrayListAddEnd`.  The only difference is in `timeMe`, which uses the two-parameter version of `add` to put the new element at index 0.  Also, we increased `endMillis` to get one additional data point.

Here are the timing results (problem size on the left, run time in milliseconds on the right):

    4000, 14
    8000, 35
    16000, 150
    32000, 604
    64000, 2518
    128000, 11555

Here's the graph of runtime versus problem size:

![alt tag](https://curriculum-content.s3.amazonaws.com/javacs/interpreting_results_figure02.small.png)

Remember that a straight line on this graph does **not** mean that the algorithm is linear.  Rather, if the runtime is proportional to `n`<sup>k</sup> for any exponent, `k`, we expect to see a straight line with slope `k`.  In this case, we expect the total time for `n` adds to be proportional to `n`<sup>2</sup>, so we expect a straight line with slope 2.   In fact, the estimated slope is 1.992, which is so close we would be afraid to fake data this good.


## Profiling `LinkedList` methods

The next exercise asked you to profile the peformance of adding new elements at the beginning of a `LinkedList`.  Based on our analysis, we expect each `add` to take constant time, because in a linked list, we don't have to shift the existing elements; we can just add a new node at the beginning.  So we expect the total time for `n` adds to be linear.

Here's our solution:

```java
	public static void profileLinkedListAddBeginning() {
		Timeable timeable = new Timeable() {
			List<String> list;

			public void setup(int n) {
				list = new LinkedList<String>();
			}

			public void timeMe(int n) {
				for (int i=0; i<n; i++) {
					list.add(0, "a string");
				}
			}
		};
		int startN = 128000;
		int endMillis = 2000;
		runProfiler("LinkedList add beginning", timeable, startN, endMillis);
	}
```

Again, we only had a make a few small changes, replacing `ArrayList` with `LinkedList` and adjusting `startN` and `endMillis` to get a good range of data.  We found that these measurements were noisier than the previous batch; here are the results:

    128000, 16
    256000, 19
    512000, 28
    1024000, 77
    2048000, 330
    4096000, 892
    8192000, 1047
    16384000, 4755

And here's the graph:

![alt tag](https://curriculum-content.s3.amazonaws.com/javacs/interpreting_results_figure03.small.png)

It's not a very straight line, and the slope is not exactly 1; the slope of the least squares fit is 1.23.  But these results indicate that the total time for `n` adds is at least approximately O(`n`), so each add is constant time.

## Adding to the end of a `LinkedList`

Adding elements at the beginning is one of the operations where we expect `LinkedList` to be faster than `ArrayList`.  But for adding elements at the end, we expect `LinkedList` to be slower.  In our implementation, we have to traverse the entire list to add an element to the end, which is linear.  So we expect the total time for `n` adds to be quadratic.

Well, it's not.

Here's the code:

```java
	public static void profileLinkedListAddEnd() {
		Timeable timeable = new Timeable() {
			List<String> list;

			public void setup(int n) {
				list = new LinkedList<String>();
			}

			public void timeMe(int n) {
				for (int i=0; i<n; i++) {
					list.add("a string");
				}
			}
		};
		int startN = 64000;
		int endMillis = 1000;
		runProfiler("LinkedList add end", timeable, startN, endMillis);
	}
```

Here are the results:

    64000, 7
    128000, 5
    256000, 21
    512000, 23
    1024000, 73
    2048000, 228
    4096000, 956
    8192000, 911
    16384000, 4539

And here's the graph:

![alt tag](https://curriculum-content.s3.amazonaws.com/javacs/interpreting_results_figure04.small.png)


Again, the measurements are noisy and the line is not perfectly straight, but the estimated slope is 1.24, which is almost exactly what we got adding elements at the beginning, and not very close to 2, which is what we expected based on our analysis.  In fact, it is closer to 1, which suggests that adding elements at the end is at least approximately linear.  What's going on?


## Doubly-linked list

Our implementation of a linked list, `MyLinkedList`, uses a singly-linked list; that is, each element contains a link to the next, and the `MyArrayList` object itself has a link to the first node.

But if you read [the documentation of `LinkedList`](https://docs.oracle.com/javase/7/docs/api/java/util/LinkedList.html), it says


> *Doubly-linked list implementation of the List and Deque interfaces. [...]  *
> *All of the operations perform as could be expected for a doubly-linked list. *
> *Operations that index into the list will traverse the list from the beginning or the end, *
> *whichever is closer to the specified index.*

If you are not familiar with doubly-linked lists, you can [read more about them here](https://en.wikipedia.org/wiki/Doubly_linked_list), but the short version is:

*  Each node contains a link to the next node and a link to the previous node.

*  The `LinkedList` object contains links to the first and last elements of the list.

So we can start at either end of the list and traverse it in either direction.  As a result, we can add and remove elements from the beginning and the end of the list in constant time!

The following table summarizes the performance we expect from `ArrayList`, `MyLinkedList` (singly-linked), and `LinkedList` (doubly-linked):

|                             | MyArrayList | MyLinkedList | LinkedList |
|-----------------------------|-----------|------------|---------------|
| add (at the end)            | **1**     | n          |   **1**       |
| add (at the beginning)      | n         | **1**      |   **1**       |
| add (in general)            | n         | n          |  n            |
| get / set                   | **1**     | n          |  n            |
| indexOf / lastIndexOf       | n         | n          |  n            |
| isEmpty / size              | 1         | 1          |  1            |
| remove (from the end)       | **1**     | n          |  **1**        |
| remove (from the beginning) | n         | **1**      |  **1**        |
| remove (in general)         | n         | n          |  n            |

The doubly-linked implementation is better than `ArrayList` for adding and removing at the beginning, and just as good as `ArrayList` for adding and removing at the end.  So the only advantage of `ArrayList` is for `get` and `set`, which require linear time in a linked list, even if it is doubly-linked.

If you know that the runtime of your application depends on the time it takes to `get` and `set` elements, an `ArrayList` might be the better choice.  If the runtime depends on adding and removing elements near the beginning or the end, `LinkedList` might be better.

But remember that these recommendations are based on the order of growth for large problems.  There are other factors to consider:

*  If these operations don't take up a substantial fraction of the runtime for your application -- that is, if your applications spends most of its time doing other things -- then your choice of a `List` implementation won't matter very much.

*  If the lists you are working with are not very big, you might not get the performance you expect.  For small problems, an quadratic algorithm might be faster than a linear algorithm, or linear might be faster than constant time.  And for small problems, the difference probably doesn't matter.

*  Also, don't forget about space.  So far we have focused on runtime, but different implementations require different amounts of space.  In an `ArrayList`, the elements are stored side-by-side in a single chunk of memory, so there is very little wasted space, and computer hardware is often faster with contiguous chunks.  In a linked list, each element requires a node with one or two links.  The links take up space (sometimes more than the cargo!), and with nodes scattered around in memory, the hardware might be less efficient.

In summary, analysis of algorithms provides some guidance for choosing data structures, but only if

1.  The runtime of your application is important,
2.  The runtime of your application depends on your choice of data structure, and
3.  The problem size is large enough that the order of growth actually predicts which data structure is better.

You could have a long career as a software engineer without ever finding yourself in this situation.

## Postscript

**CONGRATS!!!** You've reached the end of Unit 1. Stay tuned for Unit 2, which will be available on April 15!

## Resources

[Doubly linked list](https://en.wikipedia.org/wiki/Doubly_linked_list) at Wikipedia.
