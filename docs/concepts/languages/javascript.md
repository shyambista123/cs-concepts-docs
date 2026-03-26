# JavaScript

JavaScript is a high-level, interpreted programming language that is one of the core technologies of the web, alongside HTML and CSS. It enables interactive web pages and is an essential part of web applications.

## Key Features

- **Dynamic Typing**: Variables can hold values of any type
- **Prototype-based**: Object-oriented programming using prototypes
- **First-class Functions**: Functions are treated as first-class citizens
- **Event-driven**: Responds to user interactions and events
- **Asynchronous**: Supports non-blocking operations with callbacks, promises, and async/await

## Basic Syntax

### Variables

```javascript
// ES6+ variable declarations
let name = "John";        // Block-scoped, can be reassigned
const age = 30;           // Block-scoped, cannot be reassigned
var city = "New York";    // Function-scoped (legacy)

// Data types
let string = "Hello";
let number = 42;
let boolean = true;
let array = [1, 2, 3];
let object = { key: "value" };
let nullValue = null;
let undefinedValue = undefined;
```

### Functions

```javascript
// Function declaration
function greet(name) {
    return `Hello, ${name}!`;
}

// Arrow function (ES6+)
const greet = (name) => `Hello, ${name}!`;

// Function with default parameters
const greet = (name = "Guest") => `Hello, ${name}!`;

// Higher-order function
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
```

### Control Flow

```javascript
// If-else statement
if (age >= 18) {
    console.log("Adult");
} else {
    console.log("Minor");
}

// Ternary operator
const status = age >= 18 ? "Adult" : "Minor";

// Switch statement
switch (day) {
    case "Monday":
        console.log("Start of week");
        break;
    case "Friday":
        console.log("End of week");
        break;
    default:
        console.log("Midweek");
}

// Loops
for (let i = 0; i < 5; i++) {
    console.log(i);
}

array.forEach(item => console.log(item));

while (condition) {
    // code
}
```

## Object-Oriented Programming

```javascript
// ES6 Class
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    greet() {
        return `Hello, I'm ${this.name}`;
    }
    
    static species() {
        return "Homo sapiens";
    }
}

// Inheritance
class Student extends Person {
    constructor(name, age, grade) {
        super(name, age);
        this.grade = grade;
    }
    
    study() {
        return `${this.name} is studying`;
    }
}

const student = new Student("Alice", 20, "A");
console.log(student.greet());
console.log(student.study());
```

## Asynchronous JavaScript

### Callbacks

```javascript
function fetchData(callback) {
    setTimeout(() => {
        callback("Data received");
    }, 1000);
}

fetchData((data) => {
    console.log(data);
});
```

### Promises

```javascript
const fetchData = () => {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve("Data received");
        }, 1000);
    });
};

fetchData()
    .then(data => console.log(data))
    .catch(error => console.error(error));
```

### Async/Await

```javascript
async function getData() {
    try {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error("Error:", error);
    }
}

getData();
```

## Array Methods

```javascript
const numbers = [1, 2, 3, 4, 5];

// Map: Transform each element
const doubled = numbers.map(n => n * 2);

// Filter: Select elements that match condition
const evens = numbers.filter(n => n % 2 === 0);

// Reduce: Combine elements into single value
const sum = numbers.reduce((acc, n) => acc + n, 0);

// Find: Get first element that matches
const found = numbers.find(n => n > 3);

// Some: Check if any element matches
const hasEven = numbers.some(n => n % 2 === 0);

// Every: Check if all elements match
const allPositive = numbers.every(n => n > 0);
```

## ES6+ Features

### Destructuring

```javascript
// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];

// Object destructuring
const { name, age } = { name: "John", age: 30 };

// Function parameter destructuring
const greet = ({ name, age }) => `${name} is ${age} years old`;
```

### Spread Operator

```javascript
// Array spread
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];

// Object spread
const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 };
```

### Template Literals

```javascript
const name = "John";
const age = 30;
const message = `My name is ${name} and I'm ${age} years old`;
```

## DOM Manipulation

```javascript
// Select elements
const element = document.getElementById('myId');
const elements = document.querySelectorAll('.myClass');

// Modify content
element.textContent = "New text";
element.innerHTML = "<strong>Bold text</strong>";

// Modify styles
element.style.color = "blue";
element.classList.add('active');

// Event listeners
element.addEventListener('click', (event) => {
    console.log('Element clicked!');
});

// Create and append elements
const newDiv = document.createElement('div');
newDiv.textContent = "New element";
document.body.appendChild(newDiv);
```

## Common Use Cases

- **Web Development**: Interactive user interfaces
- **Server-side**: Node.js for backend development
- **Mobile Apps**: React Native, Ionic
- **Desktop Apps**: Electron
- **Game Development**: Phaser, Three.js
- **Data Visualization**: D3.js, Chart.js

## Best Practices

- Use `const` by default, `let` when reassignment is needed
- Avoid `var` in modern JavaScript
- Use arrow functions for concise syntax
- Leverage async/await for cleaner asynchronous code
- Use strict mode: `'use strict';`
- Handle errors with try-catch blocks
- Use meaningful variable and function names
- Keep functions small and focused

## Popular Frameworks and Libraries

- **React**: UI library for building user interfaces
- **Vue.js**: Progressive framework for building UIs
- **Angular**: Full-featured framework by Google
- **Node.js**: JavaScript runtime for server-side
- **Express.js**: Web framework for Node.js
- **jQuery**: DOM manipulation library (legacy)

---

- Go back to [Languages](./index.md)
- Return to [Home](../../index.md)
