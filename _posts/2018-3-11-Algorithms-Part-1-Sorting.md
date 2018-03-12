---
layout: post
title: Algorithms Part I&#58 Sorting
---

Let's talk algorithms.  No I'm not talking about those funky beats produced by Bill Clinton's former [vice president](http://i.imgur.com/Ffs0f.jpg), I'm talking about the rules that define how computer systems work and do what they do!

It is important to have an understanding of what algorithms are at an abstract level.  Often algorithms and code are mistaken for being synonymous but this is actually a conflation of the two.  Code exists to implement algorithms which, as a concept, can be entirely independent from electronics.  Algorithms are simply methods that describe the solution to a problem.  They can be as involved as [Blackjack basic strategy](https://www.blackjackapprenticeship.com/resources/blackjack-strategy-charts/) or as simple as the instructions on a can of [Campbell's chicken noodle soup](https://www.campbellsfoodservice.com/product/campbells-classic-chicken-noodle/). 

Algorithms are key to the day to day of software engineering, not just because our job is to implement and sometimes design algorithms but also because thinking "algorithmically" can make you more efficient in many aspects of your work and life.  While explicitly declaring every single step you go through to perform all of your daily tasks would not only be pedantic but incredibly limiting, I think it is important to at least consider the time and effort you spend on the processes you use to perform tasks.  When you sit down to code do you have to reconstruct your entire development environment, or is your setup ready to go?  Is the link to your favorite terminal emulator or IDE docked to the task bar, or do you have to search for it in Applications?  If you've ever wondered what lies behind historical debates like [the editor wars](https://en.wikipedia.org/wiki/Editor_war) it's the underlying concern that engineers naturally develop for the minutiae of how they go about accomplishing all parts of their work.  To the onlooker it may seem absurd that many developers actually count things like the [number of keystrokes](https://vimgolf.com/) it takes to accomplish a task, but when you're in the business of of being concerned with the extra microseconds it might take to execute an additional line of code, you begin to develop an obsessively detail oriented point of view.  Even the smallest amount of waste can add up if repeated enough times which is why it is so important we know what algorithm is the right one to use to solve a specific problem.

A classic example of a problem that we can use an algorithm to solve is the issue of [sorting](https://en.wikipedia.org/wiki/Sorting_algorithm).  Let us start with the example of a list of integers:

```
4, 15, 9, 11, 2
```

What algorithm shall we use to put these numbers in order?  The following is a simple guide to six of the most popular sorting algorithms.  We will go through and describe the implementation and trade-offs of:
### 1. [Bubble Sort](#bubble-sort)
### 2. [Insertion Sort](#insertion-sort)
### 3. [Merge Sort](#merge-sort)
### 4. [Quick Sort](#quick-sort)
### 5. [Heap Sort](#heap-sort)

**Note:** If you are unfamiliar with Big O notation I recommend reading [Gayle Laakman McDowell's Quora post on how to explain Big O in laymen's terms](https://www.quora.com/Whats-the-best-way-to-explain-big-O-notation-in-laymens-terms/answer/Gayle-Laakmann-McDowell).

## Bubble Sort

First let's have a look at Bubble Sort, as it is relatively simple to understand and is a often used as the first example of an algorithm in a CS101 course.  Here is the [pseudocode](https://en.wikipedia.org/wiki/Pseudocode) for Bubble Sort:

```
For each Integer (A) in List Do
	Swapped = false
	For each Integer (B) in List From First to Second to Last Do
		If B > B + 1 Then Swap B With B + 1, Swapped = true
	End of Loop
	If Swapped != true Then Break Loop
End of Loop
```

Using the example of our list ( ```4, 15, 9, 11, 2```) the worst case scenario would be if the list were in the exact reverse order of how we want the list sorted (```15, 11, 9, 4, 2```), the average case cannot really be represented as it's more a representation of how much time this algorithm would take on average with a random grouping of lists in all different states of sorting, the best case is if the list is already sorted (```2, 4, 9, 11, 15```).  In the worst case scenario we would need to loop through the list ```n``` times (```n``` being the number of elements in the list, in this case 5) and then for each element do an additional iteration of the ```n``` elements, leaving a runtime of ```O(n^2)```.   In the best case scenario where the elements are already sorted we would only need to iterate through the list once, leaving our run time simply as ```O(n)```.  To understand the average complexity you need only to remember that Big O drops leading constants. Since the run time of a list that is only partially sorted is always going to be some fraction of ```n^2``` and that fraction gets dropped, this leaves the average run time as ```O(n^2)```.   When speaking of algorithms ```O(n^2)``` is generally regarded as slow, well out-paced by common runtimes such as ```O(nlog(n)), O(n), O(log(n)), and O(1)``` and is faster only than the real sloths ```O(2^n) and O(n!)```.  So, so far Bubble Sort is not looking great.  It's one saving feature is its worst case space complexity which can be expressed as constant ```O(1)``` because no additional space is necessary for this algorithm.  However, there are other sorting algorithms, such as Heap Sort that achieve this feat with faster runtimes and in the age of relatively cheap memory space is usually a secondary priority anyway.  So when do we use Bubble Sort?  This algorithm is little more than a good example for learning, so it's probably best to leave it in CS101 where it belongs.

I'm going to use C for the code examples in this post because it is pretty universally understood, and is the language I am personally most comfortable with:

```c
void swapByPtr(int arr[], int a, int b){
  int tmp = arr[a];
  arr[a] = arr[b];
  arr[b] = tmp;
}

void bubbleSort(int arr[], int alen){
  int i, j;
  bool swapped;
  for(i=0; i<alen; i++){
    swapped = false;
    for(j=0; j<alen-1; j++){
      if(arr[j] > arr[j+1]){
        swapByPtr(arr, j, j+1);
        swapped = true;
      }
    }
    if(!swapped){
      break;
    }
  }
}
```

## Insertion Sort
The next popular sorting algorithm I'd like to discuss is Insertion Sort.  Which in pseudocode looks like this:

```
For each Integer (A) in Array Starting at 2nd Element Do
	Copy A into a Temporary Variable (X)
	Iterate each Integer (B) in the Array in Reverse From the Position of X
		If Element Preceding B is Less Than X Then
			Break Loop
		Copy The Element Preceding B into B's Position
	End of Loop
	Copy X into the Position Preceding B
End of Loop
```

As you can see Insertion Sort is already looking a bit deal more complicated to understand than Bubble Sort.  So how does it perform?  In the best case scenario when the list is already sorted the run time will be ```O(n)```, in the worst case scenario the list will be iterated ```n``` times and for each time it will iterate from the position of the current element to the beginning of the list.  The worst case scenario could be expressed as ```n + ... 3 + 2 + 1``` which is expressible as ```n(n+1)/2``` which is reduced to ```O(n^2)```, and the average time, being a fraction of ```n^2``` is reduced to ```O(n^2)```.  The sorting is done in place and no extra space is used so the space complexity is ```O(1)```.  Wait a minute this is starting to look familiar!  Weren't these the exact run times we just saw for Bubble Sort?  In order to understand why Insertion Sort is nearly always preferred to Bubble Sort it is important for us to dig a little deeper and look at the constant time cost of each iteration.  With Bubble Sort you are performing a costly ```swap``` operation often multiple times for the same element, but with Insertion Sort you are only performing one ```insert``` operation per element.  

Here's the C code:

```c
void insertionSort(int arr[], int alen){
  int i, j, x;
  for(i=1; i<alen; i++){
    x = arr[i];
    for(j=i-1; j>=0; j--){
      if(arr[j] <= x){
        break;
      }
      arr[j+1] = arr[j];
    }
    arr[j+1] = x;
  }
}
```

### Merge Sort
Ok now let's knock things up a notch With Merge Sort.  An extremely popular sorting algorithm that utilizes a [divide and conquer](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithm) approach.  Divide and conquer means breaking a problem down into several sub problems that are easier to solve than the whole problem itself, so think [recursion](https://en.wikipedia.org/wiki/Recursion).  Merge Sort requires the implementation of two "sub algorithms", one to break the list down into sub lists and another "merge" algorithm to merge the sub lists so that the resulting complete list is a sorted one.

```
Recursive MergeSort Start With Low Index (0), High Index (Length - 1):
	If Low Index < High Index Then
		Midpoint = ((High Index - Low Index) / 2) + Low Index
		Recurse MergeSort Left Low Index (same), High Index (Midpoint)
		Recurse MergeSort Right Low Index (Midpoint+1), High Index (same)
		Merge Low Index, Midpoint, High Index

Merge Low Index, Midpoint, High Index:
	Create Temporary List (Low) of Size (Midpoint - Low Index) + 1
	Create Temporary List (High) of Size (High Index - Midpoint)
	Copy Elements From Low Index to Midpoint in List into Low
	Copy Elements From Midpoint + 1 to High
	Low Index = 0
	High Index = 0
	List Index = 0
	Loop
		If Low at Low Index < High at High Index Then
			List at List Index = Low at Low Index
			List Index = List Index + 1
			Low Index = Low Index + 1
		Else
			List at List Index = High at High Index
			List Index = List Index + 1
			High Index = High Index + 1
		Unless Low Index < Length of Low Then
			Copy the remaining elements in High into List
			Break Loop
		Unless High Index < Length of High Then
			Copy the remaining elements in Low into List
			Break Loop
	Clean up copies
	End of Loop
```

Now we are getting much more complex than our previous examples, so let's see if it paid off.  The Merge Sort routine breaks the list down into lists of individual elements and then merges them, doubling the list sizes each time until the length of the original list is achieved.  Starting from ```1``` and doubling until we get to the length ```n``` takes ```log2(n)``` times (```1 + 2 + 4 + ... i```).  During each "merge" a copy operation is performed for each element in the list, meaning that each "merge" runs at ```O(k)```, ```k``` being the length of each sub list.  Adding all the lengths of the sub lists ```k``` together obviously equals ```n``` so this means our Merge Sort run time is ```O(n*log2(n))```.  Merge Sort has the same run time for all possible scenarios and at ```O(n*log2(n))```, Merge Sort smokes Insertion and Bubble Sort the majority of the time (the one exception being the rare best case scenario where the list is already sorted).  Unfortunately Merge Sort trades some of its space efficiency for speed.  The "merge" part of the algorithm requires the list copies Low and High which means "merge" uses ```O(k)``` space where ```k``` is the sum of the lengths of the sub lists.  Unlike time complexity when we analyze space we are not taking the total of the Big O values over the course of the entire run time of the algorithm, instead we are just trying to figure out what is the most space used at any given time.  That is why we only really need to consider the worst case space complexity, because that is how much memory we know we will have to set aside when we run the sort.  The worst case "merge" operation is the very last one where the entire size of the list is copied, and so the worst case ```k``` is equal to ```n``` and the space complexity is ```O(n)```.  ```O(n)``` space is generally regarded as being reasonable, especially in the age of [cheap memory](https://arstechnica.com/gadgets/2016/11/how-cheap-ram-changes-computing/) however there are still times when we would prefer Insertion Sort, for example to sort a List that is particularly large and costly in memory.

My C code:
```c
#include <stdlib.h> // for malloc, free
#include <string.h> // for memcpy

#define IF_NOT(expr) if(!(expr))

void * mallocExitOnFail(int len){
	void * mem;
	IF_NOT(mem = malloc(len)){
		perror("Failed to allocate mem\n");
		exit(1);
	}
	return mem;
}

void merge(int arr[], int l, int m, int h){
  int * low, * high, ll, hh, a, lLen, hLen;

  lLen = m-l+1, hLen = h-m;
  
  low = mallocExitOnFail(lLen*sizeof(int));
  high = mallocExitOnFail(hLen*sizeof(int));

  memcpy(low, &arr[l], lLen*sizeof(int));
  memcpy(high, &arr[m+1], hLen*sizeof(int));

  ll = 0, hh = 0, a = l;

  while(1){
    if(low[ll] < high[hh]){
      arr[a++] = low[ll++];
    } else {
      arr[a++] = high[hh++];
    }
    IF_NOT(ll < lLen){
      memcpy(&arr[a], &high[hh] hLen-hh);
      break;
    }
    IF_NOT(hh < hLen){
      memcpy(&arr[a], &low[ll], lLen-ll);
      break;
    }
  }

  free(low), free(high);
  }

void mergeSort(int arr[], int l, int h){
  int i, j;
  if(i < j){
    int midpoint = (h-l)/2 + l;
    mergeSort(arr, l, midpoint);
    mergeSort(arr, midpoint+1, h);
    merge(arr, l, midpoint, h);
  }
}

#define CALL_mergeSort(arr, alen)(mergeSort(arr, 0, alen-1))
```

**Note:** A quick word on macros.  As you can see I've implemented 2 preprocessor directives or "macros" for this version of Merge Sort in C.  For those of you who don't know about C macros, you cant think of them like copy and paste "templates" for code.  Before compiling your C source code into executable machine code the compiler will find all of those ```#``` signs and copy and paste code accordingly.  So for example any time I use ```IF_NOT``` in my code, the compiler will copy and paste the conditional ```if``` statement including a ```!``` in front of the chosen expression to evaluate.  This can be handy for making code more readable by obscuring unnecessary details while adding a certain level of "built-in" description.  However macros can also be incredibly [dangerous](https://stackoverflow.com/a/14041847/4996850) and should be used responsibly and in moderation.  As a rule thumb I try to use a function instead of a macro whenever I can and when I do declare macros I always make sure they are named clearly and with capitalized letters to make it obvious what they do and that they are macros.

## Quick Sort

Alright hang on for the ride because next up is Quick Sort, another divide and conquer algorithm that looks very familiar to Merge Sort but with some key differences.  Like Merge Sort, Quick Sort requires two routines including one that breaks the list down into sub lists, but for Quick Sort instead of a "merge" operation there is a "partition" operation that sorts the sub lists around a "pivot" point:

```
QuickSort Start With Low Index (0), High Index (Length - 1)
	If Low Index < High Index Then
		Pivot = Partition With Low Index, High Index
		Recurse QuickSort With Low Index, Pivot - 1
		Recurse QuickSort With Pivot + 1, High Index
		
Partition Low Index, High Index
	Current Element = List at High Index
	I Index = Low Index - 1
	For Each Element (A) From Low Index to High Index - 1 Do
		If A < Current Element Then
			I Index = I Index + 1
			Swap A, List at I Index
	End of Loop
	I Index = I Index + 1
	Swap A, List at I Index
	Return I Index
```

I like Quick Sort because in some ways it is simpler to implement than Merge Sort, if a little bit more difficult to reason about.  The worst case for this algorithm is a run time of ```O(n^2)``` if the elements are all in reverse order, otherwise this algorithm will take generally  ```O(nlog(n))``` on average and in best case scenarios.  Unlike Merge Sort, Quick Sort sorts a list in place and therefore does not require any extra space. 

The C code for Quick Sort is nice and brief:
```c
void swapByIdx(int arr[], int a, int b){
	int tmp = arr[a];
	arr[a] = arr[b];
	arr[b] = tmp;
}

int partition(int arr[], int l, int h){
	int i, j;
	for(i=l-1, j=l; j<h; j++){
		if(arr[j] < arr[h]){
			swapByIdx(arr, ++i, j);
		}
	}
	swapByIdx(arr, ++i, h);
	return i;
}

void quickSort(int arr[], int l, int h){
	if(l < h){
		int p = partition(arr, l, h);
		quickSort(arr, l, p-1);
		quickSort(arr, p+1, h);
	}
}

#define CALL_quickSort(arr, alen)(quickSort(arr, 0, alen-1))
```

So Quick Sort is nice, it is relatively easy to implement and it gives us space efficiency while maintaining most of the speed of Merge Sort, however there are still those pesky edge case scenarios where worst case Quick Sort could crawl along at a painful ```O(n^2)```.  What if we want something that uses less space with a similar run time complexity as Merge Sort?  In enters Heap Sort.

## Heap Sort

Heap Sort is super cool because it introduces us to the [heap](https://en.wikipedia.org/wiki/Heap_(data_structure)) data structure and in a way that I think is really accessible to beginners.  The heap used in a typical Heap Sort is a binary heap, meaning each node has no more than 2 children, and it is an [implicit data structure](https://en.wikipedia.org/wiki/Implicit_data_structure).  An implicit data structure in this example is an [abstract data type](https://en.wikipedia.org/wiki/Abstract_data_type) that is implemented using a much simpler data structure.  In the example of Heap Sort we take an array and then use simple arithmetic to form relations between the indices such that the elements are organized into a binary heap.  For example if we have a 0 indexed array representing our list of elements:

```c
int arr[] = { 4, 15, 9, 11, 2 };
```

Starting with the 0th index as the ```root``` of the heap we can find the elements left and right children with the following simple formulas:

```c
#define LCHILD(i)(2*i+1)
#define RCHILD(i)(2*i+2)
```

This means in our example that the left and right children of the root node are the elements at the 1th and 2th indices respectively.  So far our heap looks like:

```
	(4)
   /   \
(15)   (9)
```

Next we can descend the heap to the left child and find it's children, in this case the 3th and 4th indexed elements:

```
		(4)
	   /   \
	(15)   (9)
	/  \
  (11)	(2)
```
Now we go back up to the right child of the root and see if it has any children.  Since the values of the children of the right child are the 5th and 6th indexed elements and the lists indices only go up to 4 this node has no children and we can stop building the heap.  If at any point we wanted to find the parent of any given node we could just reverse the arithmetic:

```c
#define PARENT(i) ((i/2)-1)
```
Since we have our heap built the next step in the Heap Sort algorithm is to turn the heap into a **max heap**, meaning a heap for which every child is less than or equal to its parent, leaving the maximum element as the ```root```.  For this we can use a method called Sift Down.  We start at the element with the highest index that still has children.  To figure out which element this is we can use the ```PARENT``` formula to find the parent of the right-most element in the array, in this case, the element at ```n-1``` where ```n``` is the length of the array. We then iterate in reverse back to zero, swapping any parent nodes that are less then their children until we have a  sorted heap.  The pseudocode for Sift Down looks like this:

```
Sift Down With Sub-List Heap:
	Root = 0
	Loop While Left Child Exists
		SwapIdx = Root
		If Heap at Swap is Less Than Left Child Then
			SwapIdx = Index of Left Child
		If Right Child Exists And Heap at SwapIdx is Less Than Right Child Then
			SwapIdx = Index of Right Child
		If SwapIdx Still Equals Root Then
			Break
		Swap Elements at Root and SwapIdx
		Root = SwapIdx                                                                      
	End of Loop
```

The application of Sift Down to the array to create the Max Heap can be expressed in pseudocode as:

```
For Each Element (A) From Heap Length - 1 to 0 Do
	Sift Down With Heap = Address of Heap at (A)
End of Loop
```

Once the Max Heap has been created the first time, Heap Sort proceeds with sorting the elements by swapping the root element (the max element) with the element farthest right in the unsorted portion of the array.  The new root is now sorted back into the heap using Sift Down which takes at most an amount of time proportional to the height of the Heap, which is expressible as ```log(n)```, therefore the run time of Sift Down is ```log(n)```.  Sorting the rest of the array is done recursively in the same manner with the unsorted portion shrinking by one element each iteration.  Heap Sort performs the Sift Down operation ```n``` times which is why the run time of this algorithm is ```O(n*log(n))``` in all cases.  Since this sort is done in place the space complexity is ```O(1)```.

```c
#define LCHILD(i) (i*2+1)
#define RCHILD(i) (i*2+2)
#define PARENT(i) ((i/2)-1)

void swapByIdx(int arr[], int a, int b){
  int tmp = arr[a];
  arr[a] = arr[b];
  arr[b] = tmp;
}

void siftDown(int heap[], int hlen){
  int l, r, root, swp;
  root = 0;
  while((l = LCHILD(root)) < hlen){
    swp = root;
    if(heap[swp] < heap[l]){
      swp = l;
    }
    r = RCHILD(root);
    if(r < hlen && heap[swp] < heap[r]){
      swp = r;
    }
    if(swp == root){
      break;
    }
    swapByIdx(heap, swp, root);
    root = swp;
  }
}

void heapify(int heap[], int hlen){
  int i;
  for(i=PARENT(hlen-1); i>=0; i--){
    siftDown(&heap[i], hlen-i);
  }
}

void heapSort(int heap[], int hlen){
  int i;
  const int root = 0;
  heapify(heap, hlen);
  for(i=hlen; i>root; i--){
    siftDown(heap, i);
    swapByIdx(heap, root, i-1);
  }
}
```
