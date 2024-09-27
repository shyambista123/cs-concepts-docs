# Sorting Algorithms

This file explains different sorting algorithms.

---

Sorting algorithms are methods for arranging the elements of a list or array in a specific order, typically in ascending or descending order. Below, we will explore some of the most common sorting algorithms, their applications, and their efficiency, along with Python implementations.

## Types of Sorting Algorithms

### 1. Bubble Sort

Bubble sort is a simple comparison-based algorithm that repeatedly steps through the list, compares adjacent elements, and swaps them if they are in the wrong order. 

**Complexity:**
- Time: O(n²)
- Space: O(1)

**Python Code:**

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr
```

**Use Cases:**
- Small datasets
- Educational purposes to illustrate sorting concepts

### 2. Selection Sort

Selection sort divides the input list into two parts: a sorted and an unsorted region. It repeatedly selects the smallest (or largest) element from the unsorted part and moves it to the sorted part.

**Complexity:**
- Time: O(n²)
- Space: O(1)

**Python Code:**

```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_index = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_index]:
                min_index = j
        arr[i], arr[min_index] = arr[min_index], arr[i]
    return arr
```

**Use Cases:**
- Small datasets
- Situations where memory usage is a concern

### 3. Insertion Sort

Insertion sort builds a sorted array one element at a time. It takes each element from the unsorted list and finds its correct position in the sorted part.

**Complexity:**
- Time: O(n²)
- Space: O(1)

**Python Code:**

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1
        while j >= 0 and key < arr[j]:
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = key
    return arr
```

**Use Cases:**
- Small datasets
- Nearly sorted data

### 4. Merge Sort

Merge sort is a divide-and-conquer algorithm that divides the array into halves, sorts them, and then merges the sorted halves back together.

**Complexity:**
- Time: O(n log n)
- Space: O(n)

**Python Code:**

```python
def merge_sort(arr):
    if len(arr) > 1:
        mid = len(arr) // 2
        left_half = arr[:mid]
        right_half = arr[mid:]

        merge_sort(left_half)
        merge_sort(right_half)

        i = j = k = 0

        while i < len(left_half) and j < len(right_half):
            if left_half[i] < right_half[j]:
                arr[k] = left_half[i]
                i += 1
            else:
                arr[k] = right_half[j]
                j += 1
            k += 1

        while i < len(left_half):
            arr[k] = left_half[i]
            i += 1
            k += 1

        while j < len(right_half):
            arr[k] = right_half[j]
            j += 1
            k += 1

    return arr
```

**Use Cases:**
- Large datasets
- Stable sorting is required

### 5. Quick Sort

Quick sort is another divide-and-conquer algorithm that selects a 'pivot' element and partitions the array around it, sorting the elements on either side of the pivot.

**Complexity:**
- Time: O(n log n) on average, O(n²) in the worst case
- Space: O(log n)

**Python Code:**

```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quick_sort(left) + middle + quick_sort(right)
```

**Use Cases:**
- Large datasets
- Situations where average-case performance is critical

---

## Conclusion

Sorting algorithms are fundamental in computer science and data processing. The choice of sorting algorithm can significantly affect performance, depending on the size and nature of the data. Understanding the strengths and weaknesses of each algorithm is essential for efficient data management.

---

- Go back to [Algorithms](./index.md)
- Return to [Home](../../index.md)