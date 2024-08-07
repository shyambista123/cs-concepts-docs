### `concepts/languages/python.md`

# Python

Python is a versatile and high-level programming language known for its readability and simplicity. It is widely used in various domains, including web development, data analysis, and artificial intelligence.

## Hello, World!

The most basic program in Python is the "Hello, World!" program, which outputs the string "Hello, World!" to the console.

```python
print("Hello, World!")
```

## Variables

In Python, variables are used to store data values. You don't need to declare the variable type explicitly, as Python is dynamically typed.

### Example of Variable Assignment

```python
# Example of variable assignment
name = "Alice"   # String variable
age = 30         # Integer variable
height = 5.7     # Float variable

print(name, age, height)
```

## Data Types

Python has several built-in data types, including:
- **Strings**: Textual data, e.g., `"Hello"`
- **Integers**: Whole numbers, e.g., `42`
- **Floats**: Decimal numbers, e.g., `3.14`
- **Lists**: Ordered collections, e.g., `[1, 2, 3]`
- **Dictionaries**: Key-value pairs, e.g., `{"key": "value"}`

## Example: Simple List

Here's how to create and manipulate a list in Python:

```python
# Creating a list
fruits = ["apple", "banana", "cherry"]

# Accessing list elements
print(fruits[0])  # Outputs: apple

# Adding an element
fruits.append("orange")

# Looping through the list
for fruit in fruits:
    print(fruit)
```

## Example: Function Definition

Functions in Python are defined using the `def` keyword. Here's a simple function that adds two numbers:

```python
def add_numbers(a, b):
    return a + b

result = add_numbers(5, 3)
print(result)  # Outputs: 8
```

## Add other docs