## concepts/machine_learning/machine_learning.md

# Machine Learning

Machine Learning (ML) is a subset of artificial intelligence (AI) that focuses on building systems that can learn from data and make decisions or predictions based on that data. It involves the use of algorithms and statistical models to enable computers to improve their performance on tasks over time without being explicitly programmed.

## Hello, World! of Machine Learning

A simple introduction to ML can be through a basic example using a dataset to train a model and make predictions. Here's a basic example using the popular Python library, scikit-learn, to train a simple model:

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score

# Load the Iris dataset
data = load_iris()
X = data.data
y = data.target

# Split the data into training and test sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# Create and train a k-NN classifier
model = KNeighborsClassifier(n_neighbors=3)
model.fit(X_train, y_train)

# Make predictions
predictions = model.predict(X_test)

# Evaluate the model
accuracy = accuracy_score(y_test, predictions)
print(f"Accuracy: {accuracy:.2f}")
```

## Core Concepts

### Algorithms

Machine Learning algorithms are the core components that learn from data. Some common algorithms include:

- **Linear Regression**: Predicts a continuous value based on input features.
- **Logistic Regression**: Used for binary classification problems.
- **Decision Trees**: Makes decisions based on the features of the data.
- **k-Nearest Neighbors (k-NN)**: Classifies data based on the nearest neighbors.
- **Support Vector Machines (SVM)**: Finds the best boundary between classes.

### Data Preparation

Data preparation is a crucial step in ML and involves:

- **Data Cleaning**: Handling missing values, removing duplicates, and correcting errors.
- **Feature Engineering**: Creating new features or modifying existing ones to improve model performance.
- **Normalization/Standardization**: Scaling features to a standard range or distribution.

### Model Evaluation

Evaluating a machine learning model involves assessing its performance using metrics such as:

- **Accuracy**: The proportion of correctly classified instances.
- **Precision and Recall**: Measures of classification performance for imbalanced datasets.
- **F1 Score**: The harmonic mean of precision and recall.
- **Confusion Matrix**: A table showing true positives, false positives, true negatives, and false negatives.

## Example: Simple Linear Regression

Here’s a basic example of how to implement and use a Linear Regression model with scikit-learn:

```python
from sklearn.linear_model import LinearRegression
import numpy as np

# Sample data: Hours studied vs. Scores
X = np.array([[1], [2], [3], [4], [5]])
y = np.array([2, 4, 6, 8, 10])

# Create and train the model
model = LinearRegression()
model.fit(X, y)

# Make a prediction
predicted_score = model.predict([[6]])
print(f"Predicted score for 6 hours of study: {predicted_score[0]:.2f}")
```

This example demonstrates how to fit a linear regression model to a dataset and make predictions based on it.

