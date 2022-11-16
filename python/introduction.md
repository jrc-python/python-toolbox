# Introduction to Python <!-- omit in toc -->

- [Python Basics](#python-basics)
  - [Variables and types](#variables-and-types)
- [Python Lists](#python-lists)
  - [Manipulating lists](#manipulating-lists)
- [Functions and Packages](#functions-and-packages)
  - [Functions](#functions)
  - [Methods](#methods)
  - [Packages](#packages)
  - [Numpy](#numpy)

# Python Basics

## Variables and types

* float - real numbers
* int - integer numbers
* str - string (text)
* bool - True or False

# Python Lists

Declaration:
```python
list = [a, b, c]
family = ["liz", 1.73, "emma", 1.68, "mom", 1.71, "dad", 1.89]
```

List of lists:
```python
fam2 = [
    ["liz", 1.73],
    ["emma", 1.68],
    ["mom", 1.71],
    ["dad", 1.89]
]
```

List type:
```python
type(fam)
```
```shell
> list
```

Subsetting lists:
```python
family = ["liz", 1.73, "emma", 1.68, "mom", 1.71, "dad", 1.89

family[3] # first index is 0
# > 1.68

# last item:
fam[-1]
# > 1.89
```

List slicing:

[start : end]
| start | end |
| --- | --- |
| inclusive | exclusive |

```python
fam[3:5]
# > [1.68, 'mom']

fam[1:4]
# > [1.73, 'emma', 1.68]

fam[:4]
# > ['liz', 1.73, 'emma', 1.68]

fam[5:]
# > [1.71, 'dad', 1.89]
```

## Manipulating lists

Changing list elements

```python
fam = ["liz", 1.73, "emma", 1.68, "mom", 1.71, "dad", 1.89]

fam[7] = 1.86
fam[0:2] = ["lisa", 1.74]
# > ['lisa', 1.74, 'emma', 1.68, 'mom', 1.71, 'dad', 1.86]
```

Adding and removing elements

```python
fam + ["me", 1.79]
# > ['lisa', 1.74,'emma', 1.68, 'mom', 1.71, 'dad', 1.86, 'me', 1.79]

fam_ext = fam + ["me", 1.79]
del(fam[2])
# > ['lisa', 1.74, 1.68, 'mom', 1.71, 'dad', 1.86]
```

# Functions and Packages

## Functions

```python
fam = [1.73, 1.68, 1.71, 1.89]
max(fam)
# > 1.89

round(1.68, 1)
# > 1.7

round(1.68)
# > 2
```

## Methods

Everything is an object in Python. Objects have methods. Methods are functions that belong to objects.

Examples of list methods:

```python
fam = ['liz', 1.73, 'emma', 1.68, 'mom', 1.71, 'dad', 1.89]

fam.index("mom") # "Call method index() on fam"
# > 4

fam.count(1.73)
# > 1
```

Examples of string methods:

```python
sister = 'liz'

sister.capitalize()
# > 'Liz'

sister.replace("z", "sa")
#> 'lisa'
```

## Packages

Examples of packages available in Python:
* NumPy
* Matplotlib
* scikit-learn

```python
import numpy

numpy.array([1, 2, 3])
```
```python
import numpy as np

np.array([1, 2, 3])
```
```python
from numpy import array

array([1, 2, 3])
```

# Numpy

## Introduction

Better than lists for numerical data. Numpy arrays are faster and more compact than lists.
* Mathematical operations over collections
* Speed

Problem: limits of lists

```python
height = [1.73, 1.68, 1.71, 1.89, 1.79]
weight = [65.4, 59.2, 63.6, 88.4, 68.7]
weight / height ** 2
# > TypeError: unsupported operand type(s) for ** or pow(): 'list' and 'int'
```

Solution:NumPy
* Numeric Python
* Alternative to Python List: NumPy Array
* Calculations over entire arrays
* Easy and Fast
* Installation
  * `pip3 install numpy`

```python
import numpy as np

np_height = np.array(height)
np_weight = np.array(weight)

bmi = np_weight / np_height ** 2

# > array([21.85171573, 20.97505669, 21.75028214, 24.7473475 , 21.44127836])
```

Different types, different behavior

```python
python_list = [1, 2, 3]
numpy_array = np.array([1, 2, 3])

python_list + python_list
# > [1, 2, 3, 1, 2, 3]

numpy_array + numpy_array
# > array([2, 4, 6])
```

NumPy subsetting

```python
bmi = array([21.85171573, 20.97505669, 21.75028214, 24.7473475 , 21.44127836])

bmi[1]
# > 20.97505669

bmi > 23
# > array([False, False, False,  True, False])

bmi[bmi > 23]
# > array([24.7473475])
```

## 2D Numpy Arrays

Types of NumPy arrays

```python
import numpy as np

np_height = np.array([1.73, 1.68, 1.71, 1.89, 1.79])

type(np_height)
# > numpy.ndarray
```

2D NumPy Arrays

```python
np_2d = np.array([
    [1.73, 1.68, 1.71, 1.89, 1.79],
    [65.4, 59.2, 63.6, 88.4, 68.7]
])

np_2d.shape
# > (2, 5) # 2 rows, 5 columns
```

Subsetting

```python
np_2d[0]
# > array([1.73, 1.68, 1.71, 1.89, 1.79])

np_2d[0][2]
# > 1.71

np_2d[0, 2]
# > 1.71

np_2d[:, 1:3]
# > array([[1.68, 1.71],
# >        [59.2, 63.6]])

np_2d[1, :]
# > array([65.4, 59.2, 63.6, 88.4, 68.7])
```

## Numpy: Basic Statistics

City-wide survey
    
```python
import numpy as np

np_city = np.array([
    [1.64, 71.78],
    [1.37, 63.35],
    [1.6 , 55.09],
    ...,
    [2.04, 74.85],
    [2.04, 68.72],
    [2.01, 73.57]
])

np.mean(np_city[:, 0])
# > 1.7472

np.median(np_city[:, 0])

# > 1.75