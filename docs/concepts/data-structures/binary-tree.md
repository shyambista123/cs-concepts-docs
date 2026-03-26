# Binary Tree

A binary tree is a hierarchical data structure in which each node has at most two children, referred to as the left child and the right child. Binary trees are fundamental in computer science and are used in various applications including searching, sorting, and hierarchical data representation.

## Structure

Each node in a binary tree contains:
- **Data**: The value stored in the node
- **Left Child**: Reference to the left child node
- **Right Child**: Reference to the right child node

## Types of Binary Trees

### 1. Full Binary Tree
A binary tree where every node has either 0 or 2 children.

### 2. Complete Binary Tree
A binary tree where all levels are completely filled except possibly the last level, which is filled from left to right.

### 3. Perfect Binary Tree
A binary tree where all internal nodes have two children and all leaf nodes are at the same level.

### 4. Binary Search Tree (BST)
A binary tree where for each node:
- All values in the left subtree are less than the node's value
- All values in the right subtree are greater than the node's value

## Python Implementation

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.left = None
        self.right = None

class BinaryTree:
    def __init__(self):
        self.root = None
    
    def insert(self, data):
        if self.root is None:
            self.root = Node(data)
        else:
            self._insert_recursive(self.root, data)
    
    def _insert_recursive(self, node, data):
        if data < node.data:
            if node.left is None:
                node.left = Node(data)
            else:
                self._insert_recursive(node.left, data)
        else:
            if node.right is None:
                node.right = Node(data)
            else:
                self._insert_recursive(node.right, data)
    
    def inorder_traversal(self, node, result=None):
        if result is None:
            result = []
        if node:
            self.inorder_traversal(node.left, result)
            result.append(node.data)
            self.inorder_traversal(node.right, result)
        return result
    
    def preorder_traversal(self, node, result=None):
        if result is None:
            result = []
        if node:
            result.append(node.data)
            self.preorder_traversal(node.left, result)
            self.preorder_traversal(node.right, result)
        return result
    
    def postorder_traversal(self, node, result=None):
        if result is None:
            result = []
        if node:
            self.postorder_traversal(node.left, result)
            self.postorder_traversal(node.right, result)
            result.append(node.data)
        return result
    
    def search(self, data):
        return self._search_recursive(self.root, data)
    
    def _search_recursive(self, node, data):
        if node is None or node.data == data:
            return node
        if data < node.data:
            return self._search_recursive(node.left, data)
        return self._search_recursive(node.right, data)

# Example usage
tree = BinaryTree()
tree.insert(50)
tree.insert(30)
tree.insert(70)
tree.insert(20)
tree.insert(40)
tree.insert(60)
tree.insert(80)

print("Inorder:", tree.inorder_traversal(tree.root))    # [20, 30, 40, 50, 60, 70, 80]
print("Preorder:", tree.preorder_traversal(tree.root))  # [50, 30, 20, 40, 70, 60, 80]
print("Postorder:", tree.postorder_traversal(tree.root)) # [20, 40, 30, 60, 80, 70, 50]
```

## Tree Traversal Methods

### 1. Inorder Traversal (Left, Root, Right)
Visits nodes in ascending order for BST.

### 2. Preorder Traversal (Root, Left, Right)
Used to create a copy of the tree.

### 3. Postorder Traversal (Left, Right, Root)
Used to delete the tree.

### 4. Level Order Traversal (Breadth-First)
Visits nodes level by level.

```python
from collections import deque

def level_order_traversal(root):
    if not root:
        return []
    
    result = []
    queue = deque([root])
    
    while queue:
        node = queue.popleft()
        result.append(node.data)
        
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
    
    return result
```

## Time Complexity

| Operation | Average Case | Worst Case |
|-----------|-------------|------------|
| Search    | O(log n)    | O(n)       |
| Insert    | O(log n)    | O(n)       |
| Delete    | O(log n)    | O(n)       |
| Traversal | O(n)        | O(n)       |

## Common Applications

- **Expression Trees**: Representing mathematical expressions
- **Huffman Coding**: Data compression algorithms
- **Binary Search Trees**: Efficient searching and sorting
- **Heap Data Structure**: Priority queues
- **Syntax Trees**: Compilers and interpreters

## Advantages

- Efficient searching in BST (O(log n) average case)
- Natural hierarchical structure
- Dynamic size
- Easy to implement recursive algorithms

## Disadvantages

- Can become unbalanced, degrading to O(n) operations
- No constant-time access to elements
- More complex than linear data structures

---

- Go back to [Data Structures](./index.md)
- Return to [Home](../../index.md)
