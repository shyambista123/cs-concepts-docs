# Searching Algorithms

This file covers different searching algorithms.

---

Searching algorithms are techniques used to find a specific item in a collection of data. They can be categorized into various types, such as linear search and binary search. Below, we will explore some of the most common searching algorithms, their applications, and their efficiency, along with Python implementations.

## Types of Searching Algorithms

### 1. Linear Search

Linear search, or sequential search, is the simplest searching algorithm. It checks each element in a list one by one until the desired item is found or the list ends.

**Complexity:**
- Time: O(n)
- Space: O(1)

**Python Code:**

```python
def linear_search(arr, target):
    for index, value in enumerate(arr):
        if value == target:
            return index  # Return the index of the found element
    return -1  # Element not found
```

**Use Cases:**
- Small or unsorted datasets
- Situations where data is stored in a non-indexed format

### 2. Binary Search

Binary search is a more efficient algorithm that requires the data to be sorted beforehand. It divides the search interval in half repeatedly.

**Complexity:**
- Time: O(log n)
- Space: O(1)

**Python Code:**

```python
def binary_search(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high:
        mid = (low + high) // 2
        if arr[mid] == target:
            return mid  # Return the index of the found element
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1  # Element not found
```

**Use Cases:**
- Sorted arrays or lists
- Situations where quick search operations are needed

### 3. Jump Search

Jump search is an improvement over linear search, where you jump ahead by fixed steps and then perform a linear search within the block.

**Complexity:**
- Time: O(√n)
- Space: O(1)

**Python Code:**

```python
import math

def jump_search(arr, target):
    length = len(arr)
    jump = int(math.sqrt(length))
    prev = 0
    
    while arr[min(jump, length) - 1] < target:
        prev = jump
        jump += int(math.sqrt(length))
        if prev >= length:
            return -1  # Element not found
    
    for i in range(prev, min(jump, length)):
        if arr[i] == target:
            return i  # Return the index of the found element
    
    return -1  # Element not found
```

**Use Cases:**
- Sorted arrays where data access is costly

### 4. Interpolation Search

Interpolation search improves binary search for uniformly distributed datasets.

**Complexity:**
- Time: O(log log n) in the best case
- Space: O(1)

**Python Code:**

```python
def interpolation_search(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high and target >= arr[low] and target <= arr[high]:
        if low == high:
            if arr[low] == target:
                return low
            return -1
        
        pos = low + ((high - low) // (arr[high] - arr[low]) * (target - arr[low]))
        
        if arr[pos] == target:
            return pos  # Return the index of the found element
        if arr[pos] < target:
            low = pos + 1
        else:
            high = pos - 1
    return -1  # Element not found
```

**Use Cases:**
- Large, sorted datasets where values are evenly distributed

### 5. Exponential Search

Exponential search is useful for unbounded or infinite lists.

**Complexity:**
- Time: O(log n)
- Space: O(1)

**Python Code:**

```python
def exponential_search(arr, target):
    if arr[0] == target:
        return 0
    
    index = 1
    while index < len(arr) and arr[index] <= target:
        index *= 2
    
    return binary_search(arr[:min(index, len(arr))], target)
```

**Use Cases:**
- Unbounded arrays or scenarios where the size of the dataset is unknown

---

## Conclusion

Searching algorithms play a crucial role in data retrieval and manipulation. Choosing the right algorithm depends on the size and nature of the dataset, as well as the specific requirements of the application. Understanding the strengths and weaknesses of each algorithm is essential for optimizing performance in data-heavy environments.

---

- Go back to [Algorithms](./index.md)
- Return to [Home](../../index.md)