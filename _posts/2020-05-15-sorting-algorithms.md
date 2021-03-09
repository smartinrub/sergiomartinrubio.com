---
title: Sorting Algorithms
image: /assets/images/shubham-beeharry-223969.jpg
author: Sergio Martin Rubio
categories:
    - Algorithm
mermaid: false
layout: post
---

Sorting algorithms are used to arrange elements of a list in descending or ascending order. Sorting is a very important operation in computer science and it can be used to reduce the complexity of a problem. Some use cases are [search operations](http://sergiomartinrubio.com/articles/algorithms-to-search-through-lists-and-trees-data-structures) or [databases](http://sergiomartinrubio.com/articles/mysql-guide).

Sorting algorithms can be classified based on how simple is its implementation versus efficiency. 

## Sorting Techniques

There are two popular sorting techniques: **comparison based** and **non comparison based** algorithms (aka  or _linear sorting algorithms_).

In comparison sorting algorithms the values of the array are compared with each other to swap elements and return the sorted array. On the other hand, non-comparison sorting algorithms, values are not compared with each other, instead they rely on integer arithmetic on keys.

Comparison based sorting techniques are limited to the time complexity of $$O(nlog_2(n))$$, on the other hand, non-comparison based algorithms provide a linear time $$O(n)$$. The general problem with non-comparision algorithms is that the input data must satisfy particular requirements (e.g. each element in the array is an integer in the range 1 to k), for that reason they are not as versatile as comparison based techniques.

## Time Complexity Overview

The time complexity of the algorithms is calculated with the [Big O notation](https://sergiomartinrubio.com/articles/the-big-o-notation).

Algorithm | Type | Worst Running Time | Average Running Time 
---------|----------|----------|---------
 Bubble Sort | comparison | $$O(n^2)$$ | $$O(n^2)$$
 Selection Sort | comparison | $$O(n^2)$$ | $$O(n^2)$$
 Insertion Sort | comparison | $$O(n^2)$$ | $$O(n^2)$$
 Shell Sort | comparison | $$O(n^2)$$ | Depends on gap sequence
 Merge Sort | comparison | $$O(nlog_2(n))$$ | $$O(nlog_2(n))$$
 Heap Sort | comparison | $$O(nlog_2(n))$$ | $$O(nlog_2(n))$$
 Quick Sort | comparison | $$O(n^2)$$ | $$O(nlog_2(n))$$
 Counting Sort | non-comparison | $$O(n+k)$$ | $$O(n+k)$$
 Bucket Sort | non-comparison | $$O(n^2)$$ | $$O(n)$$
 Radix Sort | non-comparison | $$O(n)$$ | $$O(n)$$

 > Counting Sort: where `n` is the number of elements in the array and `k` is the max value of the elements

## Types

### Comparison Based

#### Bubble Sort

**Bubble sort** is probably the simplest sorting algorithms and can be written in a few lines of code. It works by simply iterating through an array and swapping the adjacent elements if they are not in order. This algorithm is called _Bubble sort_ because it keeps _bubbling up_ values to the end of the array.

Time complexity of bubble sort is $$O(n^2)$$. It's not a very efficient algorithms but is very simple to implement.

Implementation:

```java
public class BubbleSort {

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {
        for (int i = 0; i < array.length - 1; i++) {
            for (int j = 0; j < array.length - i - 1; j++) {
                if (array[j] > array[j + 1]) {
                    int aux = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = aux;
                }
            }
        }
    }

}
```

#### Selection Sort

**Selection sort** algorithm sorts arrays by finding and swapping the minimum value with the value in the current position in each iteration. This algorithm is easy to implement and is memory efficient, since it does not require an additional array for sorting. This algorithm is called _Selection sort_ because it keeps _selecting_ the smallest value.

Time complexity of bubble sort is $$O(n^2)$$. Even though _bubble sort_ and _insertion sort_ have the same time complexity in the worst case, bubble sort is outperformed most of the time by insertion sort, due to the less number of swaps required by insertion sort.

Implementation:

```java
public class SelectionSort {

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {
        for (int i = 0; i < array.length - 1; i++) {
            int min = i;
            for (int j = i + 1; j < array.length; j++) {
                if (array[j] < array[min]) {
                    min = j;
                }
            }
            int aux = array[i];
            array[i] = array[min];
            array[min] = aux;
        }
    }
}
```

#### Insertion Sort

**Insertion sort** is another simple algorithm that loops over positions in an array and in each iteration it inserts the current element into the correct position to the left of that that position.

The time complexity of insertion sort algorithm is $$O(n^2)$$, however it is more efficient than [bubble sort](http://sergiomartinrubio.com/articles/sorting-algorithms#bubble-sort) and [selection sort](http://sergiomartinrubio.com/articles/sorting-algorithms#selection-sort) because you can stop inner loop earlier, after you found correct position for the current element, so if each element is about half in order it will take $$\frac{n^2}{4}$$ which is better than the worst case for bubble and selection sort algorithms $$\frac{n^2}{2}$$.

$$T(n)=\sum_{i=1}^{n-1}\frac{i}{2}$$

$$T(n)=\frac{1}{2}(1 + 2 + 3 ...+(n - 1))$$

$$T(n)=\frac{(n-1)n}{4} \approx O(n^2)$$

This algorithm has a simple implementation that can be written in Java language as follows:

```java
public class InsertionSort {

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {
        for (int i = 1; i < array.length; i++) {
            int currentValue = array[i];
            int j = i;

            while (j >= 1 && array[j - 1] > currentValue) {
                array[j] = array[j - 1];
                j--;
            }
            array[j] = currentValue;
        }
    }
}
```

#### Shell Sort

**Shell sort** is nothing but [insertion sort](http://sergiomartinrubio.com/articles/sorting-algorithms#insertion-sort) by using gap. For particular elements that are far apart from its correct position it allows to jump more than one step a time so it can reach the proper destination in fewer exchanges.

This algorithm stars with a big enough gap so elements that are far apart are exchanged. There is incremental sequence for the gap. During each gap iteration a subset of the array is sorted.

The time complexity of Shell sort is $$O(n^2)$$ but it performs better than any other $$O(n^2)$$ algorithm. Average Case Time complexity depends on gap sequence.

Implementation in Java:

```java
public class ShellSort {

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {
        // The gap is divided by 2 in each iteration
        for (int gap = array.length / 2; gap > 0; gap /= 2) {
            // A portion of the array is sorted
            for (int i = gap; i < array.length; i++) {
                int aux = array[i];
                int j = i;

                // Compare both ends of the array subset
                // In the first iteration array[0] > array[array.length/2]
                while (j >= gap && array[j - gap] > aux) {
                    array[j] = array[j - gap];
                    j -= gap;
                }
                array[j] = aux;
            }
        }
    }
}
```

#### Merge Sort

**Merge sort** is a _divide and conquer algorithm_. An array is divided in two halves by recursion and then it sorts and merges the two halves.

>Divide-and-conquer algorithm is based on breaking down a problem into sub-problems until they became simple enough to be solved directly.

The time complexity of merge sort algorithm is $$O(nlog_2(n))$$.

$$
T(n)=\Biggr\{\begin{eqnarray} O(1)\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;  if\;\; n = 1  \\
2T(n/2)+O(n) \;\;\;if\;\; n > 1
\end{eqnarray}
$$

$$
T(n)=\Biggr\{\begin{eqnarray} c\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;  if\;\; n = 1  \\
2T(n/2)+cn \;\;\;if\;\; n > 1
\end{eqnarray}
$$


where c represents the time required to solve the problem of size 1 as well as the time per array element of the divide and combine steps.

Each level below the top one will have $$2^i$$ nodes, and each with a cost of $$c(n/2^i)$$. Therefore, the top level will have a cost of:

$$cn=2^ic\biggl(\frac{n}{2^i}\biggl)$$

where $$i$$ is the level. The bottom level has n nodes with a cost of c each one, so the total cost is $$cn$$.

$$T(1)=\biggl(\frac{n}{2^i}\biggl)=1$$

$$i=log_2(n)$$

The tree will have log $$log_2(n)+1$$ levels each costing cn. Then:

$$cn(log_2(n) + 1) = O(nlog_2(n))$$

Java Implementation:

```java
public class MergeSort {

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {
        if (array.length > 1) {
            int[] left = new int[array.length / 2];
            int[] right = new int[array.length - left.length];
            System.arraycopy(array, 0, left, 0, left.length);
            System.arraycopy(array, left.length, right, 0, right.length);

            sort(left);
            sort(right);
            merge(array, left, right);
        }

    }

    private void merge(int[] result, int[] left, int[] right) {
        int indexLeft = 0;
        int indexRight = 0;
        int indexResult = 0;

        while (indexLeft < left.length && indexRight < right.length) {
            if (left[indexLeft] < right[indexRight]) {
                result[indexResult] = left[indexLeft];
                indexLeft++;
            } else {
                result[indexResult] = right[indexRight];
                indexRight++;
            }
            indexResult++;
        }

        System.arraycopy(left, indexLeft, result, indexResult, left.length - indexLeft);
        System.arraycopy(right, indexRight, result, indexResult, right.length - indexRight);
    }

}

```

#### Heap Sort

**Heap sort** is an efficient sorting algorithm that uses two types of data structures, [arrays](http://sergiomartinrubio.com/articles/introduction-to-java-collections#arrays) and [trees](http://sergiomartinrubio.com/articles/introduction-to-java-collections#trees). The type of tree used by this algorithm is max-heap tree, which essentially puts the largest elements at the top of the tree.

{% include elements/figure.html image="https://lh3.googleusercontent.com/9YJJZ8aZ62fmwj_7_pHSRP_NGv8qTjgfugf1u7o2Zdxs8wRmTHEvJX3eZYOSoQuKrDgIo-RzJ6FyoBfmmWWx1NRDneAKRJsVCCd7c_wqMPzSE_Ho2oF4U7QsMFiHNtrhlSJHbL1cHg=w800" caption="Heap Sort Initial Array" %}

{% include elements/figure.html image="https://lh3.googleusercontent.com/sxKVm3rm4sLqGsRwbqY41Rpd5KZSslg6r__Sm3La50sTYIYeaxMj5dT9V_GLLcJ8RHT4OU_qgN_RaAfXd2mqX7rtCeWiksGVGAJapFk7yjbuXC_8RypuAT5y-Pp51zQccovOl3R4LA=w800" caption="Heap Sort - heapify" %}

Time complexity of heap sort is $$O(nlog_2(n))$$.

Java Implementation:

```java
public class HeapSort {

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {
        buildMaxHeapTree(array);
        // Once the heap tree is built the array is ordered
        for (int lastNodeIndex = array.length - 1; lastNodeIndex > 0; lastNodeIndex--) {
            int aux = array[0];
            // the greatest value in on the first node of the tree,
            // so it's now put at the end of the array
            array[0] = array[lastNodeIndex];
            array[lastNodeIndex] = aux;
            // put greatest value at the root again
            heapify(array, lastNodeIndex, 0);
        }
    }

    // O(n)
    void buildMaxHeapTree(int[] array) {
        int treeLengthExcludingLastRow = array.length / 2 - 1;
        // Go through all the node except for the last row of the tree
        for (int nodeIndex = treeLengthExcludingLastRow; nodeIndex >= 0; nodeIndex--) {
            heapify(array, array.length, nodeIndex);
        }

    }

    // O(log n)
    void heapify(int[] array, int arrayLength, int nodeIndex) {
        int largest = nodeIndex;
        // In a tree the left node of any node is at: the double of the current position plus one
        int left = nodeIndex * 2 + 1;
        // In a tree the left node of any node is at: the double of the current position plus two
        int right = nodeIndex * 2 + 2;

        // The left child is greater than the current node?
        if (left < arrayLength && array[left] > array[largest]) {
            largest = left;
        }

        // The right child is greater than the current node?
        if (right < arrayLength && array[right] > array[largest]) {
            largest = right;
        }

        // if any of the children nodes are greater than the current node
        // the nodes are swapped
        if (largest != nodeIndex) {
            int aux = array[nodeIndex];
            array[nodeIndex] = array[largest];
            array[largest] = aux;
            // keep moving to the child
            heapify(array, arrayLength, largest);
        }

    }
}
```

#### Quick Sort

Quick sort is a divide and conquer algorithm which is based on partitions. It uses recursion in order to split an array in halves.

How does it work?

1. Partition the original array and select the first pivot element.
2. You usually choose either the rightmost element or leftmost element in each subarray as the pivot.
3. The array is split in to two halves, one with values larger than the pivot and the other with values smaller than the pivot.
4. The quick sort method is called recursively for both halves in every partition.

This algorithm can be improved by calculating the median of a few randomly chosen elements and use this median as the pivot. If you choose a large number of elements at random the median will improve but it will increase the time to calculate the median.

The time complexity of quick sort in the worst case is $$O(n^2)$$ however the average time is $$O(nlog_2(n))$$.

Java implementation:

```java
public class QuickSort {

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {
        quickSort(array, 0, array.length - 1);
    }

    private void quickSort(int[] array, int lowerBound, int upperBound) {
        int pivot;

        if (upperBound > lowerBound) {
            pivot = partition(array, lowerBound, upperBound);
            quickSort(array, lowerBound, pivot - 1);
            quickSort(array, pivot + 1, upperBound);
        }
    }

    private int partition(int[] array, int lowerBound, int upperBound) {
        int pivotElement = array[lowerBound];
        int leftIndex = lowerBound;
        int rightIndex = upperBound;

        while (leftIndex < rightIndex) {
            while (array[leftIndex] <= pivotElement) {
                leftIndex++;
            }

            while (array[rightIndex] > pivotElement) {
                rightIndex--;
            }

            if (leftIndex < rightIndex) {
                int aux = array[leftIndex];
                array[leftIndex] = array[rightIndex];
                array[rightIndex] = aux;
            }
        }

        array[lowerBound] = array[rightIndex];
        array[rightIndex] = pivotElement;
        return rightIndex;
    }
}
```

### Non-Comparison Based

#### Counting Sort

**Counting sort** is an integer sorting algorithm that is only applicable for countable elements (real numbers are excluded). This algorithm requires an array where the values range from `0` to `k`, where `k` is an _integer_. Counting sort will place the element into the correct position of the output array.

The time complexity of counting sort is $$O(n+k)$$ where `n` is the number of elements in the array and `k` is the max value of the elements. As you can see counting sort has a better performance than any of the comparison based algorithms, however it requires more memory allocation. This algorithm is recommended when $$k \approx O(n)$$.

Implementation steps:

1. Find largest value in array.
2. Create and initialize to zero an auxiliary array to store the count.
3. Store count of each element of the original array.
4. Update count array with the cumulative count.
5. Find the index of each element of the original array in the count array and place the elements in the sorted array.
6. Copy elements from the sorted array to the original array.

```java
public class CountingSort {

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {

        // find larges value in array
        int max = array[0];
        for (int i = 1; i < array.length; i++) {
            if (array[i] > max) {
                max = array[i];
            }
        }

        int[] count = new int[max + 1];
        // initialize output array with zero values given max value
        for (int i = 0; i <= max; i++) {
            count[i] = 0;
        }

        // store count of each element
        for (int value : array) {
            count[value]++;
        }

        // store cumulative count of each element
        for (int i = 1; i <= max; i++) {
            count[i] += count[i - 1];
        }

        int[] sorted = new int[array.length + 1];
        // find the index of each element of the original array
        // in the count array and place the elements in the sorted array
        for (int i = array.length - 1; i >= 0; i--) {
            sorted[count[array[i]] - 1] = array[i];
            count[array[i]]--;
        }

        // copy sorted array to original array
        for (int i = 0; i < array.length; i++) {
            array[i] = sorted[i];
        }
    }
}
```

#### Bucket Sort

Bucket sort is used when the input is expected to be uniformly distributed, it assumes that the input is generated randomly.

Bucket sort puts the elements into `n` intervals or buckets of the same size and then distributes the values into the buckets. Since the input is uniformly distributed we expect that each bucket contains similar amount of elements. Then we simply need to sort the values in each bucket and finally concatenate the buckets in order.

Time complexity of bucket sort algorithm is $$O(n^2)$$ in the worst case and $$O(n)$$ on average.

Java implementation:

```java
public class BucketSort {

    private static final int BUCKET_SIZE = 2;

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {

        // calculate min and maximum values
        int minValue = array[0];
        int maxValue = array[0];
        for (int value : array) {
            if (value < minValue) {
                minValue = value;
            }
            if (value > maxValue) {
                maxValue = value;
            }
        }

        // initialize buckets
        int bucketCount = (maxValue - minValue) / BUCKET_SIZE + 1;
        List<Integer>[] buckets = new List[bucketCount];
        for (int i = 0; i < bucketCount; i++) {
            buckets[i] = new ArrayList<>();
        }

        // distribute values in buckets
        for (Integer value : array) {
            buckets[(value - minValue) / BUCKET_SIZE].add(value);
        }

        // sort buckets and concatenate
        int currentIndex = 0;
        for (List<Integer> listBucket : buckets) {
            Integer[] arrayBucket = new Integer[listBucket.size()];
            arrayBucket = listBucket.toArray(arrayBucket);
            insertionSort(arrayBucket);
            for (Integer value : arrayBucket) {
                array[currentIndex++] = value;
            }
        }

    }

    public void insertionSort(Integer[] array) {
        for (int i = 1; i < array.length; i++) {
            int currentValue = array[i];
            int j = i;

            while (j >= 1 && array[j - 1] > currentValue) {
                array[j] = array[j - 1];
                j--;
            }
            array[j] = currentValue;
        }
    }

}
```

#### Radix Sort

Radix sort is another non comparison based algorithm which is based on sorting digits.

How does it work?

1. Get the largest number.
2. Select the least significant digit (rightmost digit), this means that in the first iteration all the elements will be sorted by the rightmost digit, then by the second rightmost digit and so on and so forth until the most significant digit. 
3. During each iteration the values are sorted with a stable sort algorithm like the  Sort](http://sergiomartinrubio.com/articles/sorting-algorithms#counting-sort).

> Stable sorting algorithm: elements with the same value remain in the same position.

The time complexity of radix sort (if we use counting sort as the sorting algorithm for each digit) is $$O(n)$$:

$$O(n+k)$$

where `k` is the max value of the elements. Counting sort is called `d` times, where `d` is the number of digits

$$O(d(n+k))$$

as `d` is constant and

$$k=O(n)$$

then

$$O(d(n+k)) \approx O(n)$$

Java implementation:

```java
public class RadixSort {

    /**
     * Sort an array
     *
     * @param array to be sorted
     */
    public void sort(int[] array) {
        int maxValue = findMax(array);

        for (int multiplier = 1; maxValue / multiplier > 0; multiplier *= 10) {
            countingSort(array, multiplier, maxValue);
        }
    }

    private int findMax(int[] array) {
        int max = array[0];
        for (int i = 1; i < array.length; i++) {
            if (array[i] > max) {
                max = array[i];
            }
        }
        return max;
    }

    private void countingSort(int[] array, int position, int maxValue) {
        int[] aux = new int[maxValue + 1];
        int[] output = new int[array.length];

        for (int i = 0; i < array.length; i++) {
            aux[(array[i] / position) % 10]++;
        }

        for (int i = 1; i < maxValue; i++) {
            aux[i] = aux[i] + aux[i - 1];
        }

        for (int i = array.length - 1; i >= 0; i--) {
            output[aux[(array[i] / position) % 10] - 1] = array[i];
            aux[(array[i] / position) % 10]--;
        }
        for (int i = 0; i < array.length; i++) {
            array[i] = output[i];
        }
    }

}
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-sorting-algorithms.git" text="Examples" %}
</p>
