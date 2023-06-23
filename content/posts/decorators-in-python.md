---
title: "decorators in python with examples"
date: 2023-06-23T17:05:00+00:00 # actually 19:05 CEST
# weight: 1
# aliases: ["/first"]
tags: ["python", "decorators"]
author: "mrturkmen"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "decorators in python"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "" # image path/url
    alt: " " # alt text
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/edit/master/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---



Python has some excellent features which provide you confidence. One of them is decorators. Decorators are used to modify the behaviour of a function or a class. In this article, we will learn how to use decorators in Python.

## What are decorators?

In programming, decorators are functions that enhance the functionality of another function by receiving it as input, modifying it, and then returning a new function. This process is known as metaprogramming, as one part of the program modifies another part during compilation.

## How to use decorators?

In Python, functions are first-class objects. It means that functions can be passed around and used as arguments, just like any other object (string, int, float, list, and so on). Consider the following examples:

--- 
```python
# this function takes another function as an argument, adds functionality and returns another function
def uppercase_decorator(func):
    def wrapper():
        result = func()
        return result.upper()
    return wrapper

@uppercase_decorator # using decorator 
def say_hello():
    return "Hello, world!"

print(say_hello())  # Output: HELLO, WORLD!
```

--- 
Another example for logging: 

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print('Logging execution')
        func(*args, **kwargs)
        print('Done logging')
    return wrapper

@logger
def sample():
    print('-- Inside sample function')

sample()
```

Output of the above code is:

```bash
Logging execution
-- Inside sample function
Done logging
```
--- 

Another example with classes: 

```python
class LogDecorator:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print(f"Calling function: {self.func.__name__}")
        return self.func(*args, **kwargs)

@LogDecorator
def add_numbers(x, y):
    return x + y

result = add_numbers(3, 5)  # Output: Calling function: add_numbers
print(result)  # Output: 8
``` 

--- 

Another example for authentication: 

```python
def login_required(func):
    def wrapper(*args, **kwargs):
        if is_user_logged_in():
            return func(*args, **kwargs)
        else:
            raise Exception("User must be logged in to access this resource.")
    return wrapper

@login_required
def restricted_function():
    return "This function requires authentication."

# Usage
restricted_function()
```
--- 
Another example for processing data with threads and decorators: 

```python

import concurrent.futures

def concurrent_execution(num_threads):
    def decorator(func):
        def wrapper(*args, **kwargs):
            # Create a ThreadPoolExecutor with the specified number of threads
            with concurrent.futures.ThreadPoolExecutor(max_workers=num_threads) as executor:
                # Submit the function to the executor as a concurrent task
                future = executor.submit(func, *args, **kwargs)
                # Wait for the task to complete and retrieve the result
                result = future.result()
                return result
        return wrapper
    return decorator

@concurrent_execution(num_threads=4)
def process_data(data):
    # Perform data processing tasks (e.g., data transformation, aggregation, analysis)
    # This function is executed concurrently by multiple threads
    # ...
    return processed_data

# Usage
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

@concurrent_execution(num_threads=4)
def process_data(data):
    # Perform data processing tasks (e.g., data transformation, aggregation, analysis)
    # This function is executed concurrently by multiple threads
    # ...
    return processed_data

# Usage
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

@concurrent_execution(num_threads=4)
def process_data(data):
    # Perform data processing tasks (e.g., data transformation, aggregation, analysis)
    # This function is executed concurrently by multiple threads
    # ...
    return processed_data

# Usage
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Decorated function will be executed concurrently by four threads
result = process_data(data)
print(result)
```

## References 

- [Python Decorators-Documentation](https://peps.python.org/pep-0318//)
- [Datacamp Decorators](https://www.datacamp.com/tutorial/decorators-python)
- ChatGPT :')
