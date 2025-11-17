# 7. Testing
Testing implies verifying a piece of code (or part of it) for its behavior—be it functional, non-functional, or quality behavior. One obvious way of testing one’s code is to run it several times with various inputs—often testing boundary conditions, equivalence classes etc.—and observe the output. This form of testing requires the tester to correctly identify input scenarios in which a code can behave erroneously. However, this form of manual testing is laborious, and one may not be able to imagine and test every possible scenario. 
Python provides several methods to test code’s behavior. In the following, we discuss some of the most common methods. 

## 7.1. Manual testing 
The simplest form of testing involves repeatedly running the code with varying inputs to test if code behaves as expected. For example, consider the following graph storage format converter. One way to test if it works correctly is by repeatedly executing the file with various inputs and note the behavior. 
```python
def graph_convert(graph):
    '''
    This functions converts a graph to an alternative format. If the input is 
    an adjencecy matrix (2-D list), it converts it to an adjencecy dictionary. 
    If the input is an adjencecy dictionary, it is converted to an adjencecy 
    matrix. The result is returned to the caller.
    '''

    if isinstance(graph, dict):
        # if input graph is dict:    
        adj_mat = [
            [
                1 if j in graph.get(i, []) else 0
                for j in range(len(graph))
            ]
            for i in range(len(graph))
        ] 
        return adj_mat
    
    elif isinstance(graph, list):
        # if input graph is matrix:  
        adj_dict = {
            row_no: [ j for j in range(len(graph))
                     if graph[row_no][j] ]
            for row_no in range(len(graph))
        }
        return adj_dict
    
    else:
        # the input is not in expected format
        return None
```
However, to test it systematically, we need to use some testing framework. 
 
## 7.2 Testing frameworks in Python 
A testing framework in python refers to a systematic way of automating code testing. It is typically provided by libraries or modules, some of which are already built-in in python. A few popular examples include `unittest` (built-in), `pytest`, and `hypothesis`. 

## 7.3. The `unittest` library 
To test the `graph_converter` function, we develop a **test case** with `unittest` library in this section. A test case is a class with testing code which inherits from `unittest.TestCase` base class provided by the `unittest` framework. A **test case** class contains some **test methods** which test the code by calling appropriate functions, creating objects, and `assert`-ing some expressions. A **test suite** aggregates one or more test classes, and test methods to form a meaningful testing group. 
To test the `graph_converter` we write `TestGraphConvert` test case. This class is saved in `test_graph.py` which resides in the same project directory. The test case class contains a test method `test_dict2mat`. Please note the naming conventions here. The class, the methods, and the file all start with `test` prefix. It is customary and facilitates automation of test case collection/execution by the **test runner**. 
```python
# file: test_graph.py
import unittest 
import graph_converter as gc 

class TestGraphConvert(unittest.TestCase):
    '''This class tests the behaior of graph convertor function.'''
    def test_dict2mat(self):
        '''This methods tests dictionary to matrix conversion'''
        self.adj_dict = {0: [1, 2], 1: [0, 2], 2: [1, 0, 3], 3: [2]}
        self.expected = [[0, 1, 1, 0], [1, 0, 1, 0], [1, 1, 0, 1], [0, 0, 1, 0]]
        
        self.returned_mat = gc.graph_convert(self.adj_dict)

        self.assertIsInstance(self.returned_mat, list, 
                        f'Returned output is not a matrix')
        self.assertEqual(self.returned_mat, self.expected, 
                         f'Returned matrix is not as expected: {self.expected}')
```
The `test_graph.py` imports `unittest` and the `graph_converter` which it needs to test. We then write a simple test method `test_dict2mat`. This test method calls our target function `graph_convert` and passes an adjacency dictionary `self.adj_dict`. We also create `self.expected` to store what we expect as the result of our function call. Once `graph_convert` returns a value, we *assert* two tests. First, we check if the returned value has `<list>` as type. Second, we assert that the returned value should match what we expect. The message in assert statements is displayed if an assertion fails.  
The `unittest` has its own pool of **assertion** methods which you shall read about in the official documentation. A typical asser method call is of the form\
`<assert_method>(<recieved data to test>, <expected data to match>, <message (optional) to display if the test fails>)`\
The test cases can run in several ways. We recommend that you initially run it with verbose switch `-v` which brings more information, though You can run your testsuite without it. The output from our test run shows that the `test_dict2mat` test method passed. The output also shows how much time was consumed.
```sh
$ python3 -m unittest -v
test_dict2mat (test_graph.TestGraphConvert.test_dict2mat) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
``` 
Expanding the previous example, we now want to test the functionality for converting an adjacency matrix to an adjacency dictionary, and also want to test if  `graph_convert`  behaves as expected on invalid parameters. These behaviors are tested by test methods `test_mat2dict` and `test_invalid_input` respectively. 
```python
# file: test_graph.py
import unittest 
import graph_converter as gc 

class TestGraphConvert(unittest.TestCase):
    '''This class tests the behaior of graph convertor function.'''
    def test_dict2mat(self):
        '''This methods tests dictionary to matrix conversion'''
        self.adj_dict = {0: [1, 2], 1: [0, 2], 2: [1, 0, 3], 3: [2]}
        self.expected = [[0, 1, 1, 0], [1, 0, 1, 0], [1, 1, 0, 1], [0, 0, 1, 0]]
        
        self.returned_mat = gc.graph_convert(self.adj_dict)

        self.assertIsInstance(self.returned_mat, list, 
                        f'Returned output is not a matrix')
        self.assertEqual(self.returned_mat, self.expected, 
                         f'Returned matrix is not as expected: {self.expected}')        
    
    def test_mat2dict(self):
        '''This methods tests matrix to dictionary conversion'''
        self.adj_mat = [[0, 1, 1, 0], [1, 0, 1, 0], [1, 1, 0, 1], [0, 0, 1, 0]]
        self.expected = {0: [1, 2], 1: [0, 2], 2: [1, 0, 3], 3: [2]}

        self.returned_dict = gc.graph_convert(self.adj_mat)
        
        self.assertIsInstance(self.returned_dict, dict, 
                        f'Returned output is not a dictionary')
        self.assertEqual(self.returned_dict, self.expected, 
                         f'Returned dictionary is not as expected: {self.expected}')
        
    def test_invalid_input(self):
        '''This methods tests if converter function returns None with invalid input'''
        self.input = 'something else than a matrix of a dictionary'
        self.expected = None 

        self.returned_val = gc.graph_convert(self.input)

        self.assertIsNone(self.returned_val, "Expected None, but recieved something")
        
if __name__=='__main__':
    unittest.main()
```
The `unittest.main` provides entry point to the unittest framework. When this function is called, it scans all your classes *inherited* from the `unittest.TestCase` superclass. It then goes on to find all methods in all such classes which have a *test* prefix in their names. All those methods are then executed in alphabetical order. 
```sh
$ python3 -m unittest -v
test_dict2mat (test_graph.TestGraphConvert.test_dict2mat)
This methods tests dictionary to matrix conversion ... ok
test_invalid_input (test_graph.TestGraphConvert.test_invalid_input)
This methods tests if converter function returns None with invalid input ... ok
test_mat2dict (test_graph.TestGraphConvert.test_mat2dict)
This methods tests matrix to dictionary conversion ... FAIL

======================================================================
FAIL: test_mat2dict (test_graph.TestGraphConvert.test_mat2dict)
This methods tests matrix to dictionary conversion
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\Code\DAT690\DAT690-Practice\graphs\test_graph.py", line 28, in test_mat2dict
    self.assertEqual(self.returned_dict, self.expected,
    ~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                     f'Returned dictionary is not as expected: {self.expected}')
                     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError: {0: [1, 2], 1: [0, 2], 2: [0, 1, 3], 3: [2]} != {0: [1, 2], 1: [0, 2], 2: [1, 0, 3], 3: [2]}
- {0: [1, 2], 1: [0, 2], 2: [0, 1, 3], 3: [2]}
?                              ---

+ {0: [1, 2], 1: [0, 2], 2: [1, 0, 3], 3: [2]}
?                            +++
 : Returned dictionary is not as expected: {0: [1, 2], 1: [0, 2], 2: [1, 0, 3], 3: [2]}

----------------------------------------------------------------------
Ran 3 tests in 0.001s

FAILED (failures=1)
```
Note that the failure occurs in `test_mat2dict` method, where the returned dictionary does not match with our expected dictionary. If you look closely, the problem arises in the following part of returned dictionary\
`… 2: [0, 1, 3] …`
while expected dictionary is as below\
`… 2: [1, 0, 3] …`\
This difference of order in the list may not be significant in the context of a graph. That means, in a graph’s syntactical sense, these values mean the same edges between same vertices. However, in a pythonic context they are different due to lists being **ordered structures**. Therefore, you should be careful about such situations. 

### Interpreting the output: 
A ‘ **.** ’ in the test output indicates a passed test. An ‘ **F** ’ appears for a failed test. The failed test also shows further information about cause of the failure, as well as displays the message which programmer has provided in the test case. Sometimes, you may see an ‘ **E** ’ which means the test method has encountered an error. The error message is typically verbose. 

### Some Good things to know: 
If you need to test the behavior of a class, you need to import that class in your test file and make its object for testing. You can make this test object separately for each test method, or you can make one object in `setUp`  method which can then be used by several test methods, as shown in the example code below. 
```python
import unittest
from calc import Calc 

class TestDiff(unittest.TestCase):
    '''test case for testing our simple calculator class.'''

    def setUp(self):
        self.testObj = Calc(55, 25)

    def test_sum(self):        
        self.assertEqual(self.testObj.get_sum(), 80, "incorrect addition result")
    
    def test_dif(self):
        self.assertEqual(self.testObj.get_difference(), 30, "incorrect subtraction result")    
```
Each approach has its own merits. For example, when test methods work with their own objects, they work independently of each other. One test methods does not affect another test method’s execution. On the other hand, one shared object made in `setUp` saves memory, and may be useful in situations where multiple behaviors successively depend upon each other. The `setUp` and `tearDown` methods in a test case are used to setup the testing environment, and clean it up after the test, respectively. 

### The difference between `assert` and `<assert methods>`
Python provides a builtin 'assert' keyword which is used to ascertain if a condition is true. If the condition becomes false, it raises an `<AssertionError>`
```python
def add_integers(x, y):
    return x + y 
assert add_integers(10, 15) == 5 
```

```sh
$ python3 .\assert.py
Traceback (most recent call last):
  File ".\assert.py", line 2, in <module>
    assert add_integers(10, 15) == 5
           ^^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError
```
The `assert` keyword is quite helpful for testing and debugging. However, testing frameworks have implemented their own versions of *assertions*, typically in form of asseertion methods. The `unittest` assertion methods are one such example, two of which you have seen in previous examples, namely `self.assertEqual` and `self.assertIsNone`. For a more detailed range of `unittest` assert methods, refer to [unittest library official documentation](https://docs.python.org/3/library/unittest.html)
 
## 7.4. The pytest library :TODO 


## 7.5. Property-based testing 
In the previous example, where we tested the behavior of `graph_convert` on invalid input, you might have identified that the test case coverage was minimal. It only checked one type of invalid input, for example a `<str>`. However, there is a wide range of other invalid inputs which we have not yet tested, for example integers, lists, or malformed dictionaries.\
Similarly, we have checked valid inputs via `test_mat2dict` and `test_dict2mat`. But we are not certain if our code works on all different kinds of valid inputs, for example, on a **null graph** (fully disconnected graph with no edges between vertices).\
Writing test cases for an all-encompassing test input range, or for a wider coverage of scenarios, is tiresome; or not possible at all. In such situations, property-based testing proves to be useful. Property-based testing is based on *Fuzzing* idea, which means randomly generating large amounts of test data with the help of software. These test data often include boundary cases, invalid inputs, as well as valid inputs.

### 7.5.1 The hypothesis library
We have just discovered that it is significantly difficult for programmers/testers 
 1) to generate a large number of valid input data values
 2) to identify all possible invalid input classes, and to generate all possible invalid input data instances
 
Even if such inputs have been generated, it is difficult to automate the function/method call with this large array of inputs.  
The hypothesis library solves this problem with two simple mechanisms, namely `given` and `strategies`. 
1) `strategies`: generates a range of random data based on some predefined strategy. A few noteable strategies include generating integers, floats, text, lists, matrices, and dictionaries.  
2) `given`: in simple terms, it passes (*gives*) randomly generated data (*by strategies*) to the function we need to test and calls it repetedly until all random data instances are passed. Pythonically, this mechanism is implemented with `<decorators>`. The `<decorators>` transform (*enhance*) the behavior of a function. In this case `given` decorator enhances the behavior of target function in a way that it is called over and over again with the strategy-generated random input. (more on `<decorators>` soon)


#### 7.5.2. Property-based testing: first example
Graph algorithms are very subtle, and it is easy to forget testing them with all relevant kinds of cases.Thus, it is important to test them using **property-based testing**, which is made available by the `hypothesis` library:

* <https://hypothesis.works/> (main page)
* <https://hypothesis.readthedocs.io/> (documentation)



The Hypothesis documentation starts with a quick start example, which we have modified just a bit:

```python
from hypothesis import given, strategies as st

@given(st.integers(), st.integers())
def test_ints_are_commutative(x, y):
    assert x - y == y - x

test_ints_are_commutative()
```
The function we want to test here is `test_ints_are_commutative(x, y)`. It takes two integers as parameters. The statement `@given(st.integers(), st.integers())` transformers the behvior of `<test_ints_are_commutative>`. The first strategy `<st.integers()>` generates random integers for `<parameter x>` and the second strategy `<st.integers()>` generates random integers for `<parameter y>`. The `<@given>` decorators makes it possible to call `test_ints_are_commutative(x, y)` over and over again with randomly generated input. Hence, the test case is run a large number of times with varying inputs. 

When we run this (written in the file `htest.py`), we get

```sh
$ python3 htest.py 
Falsifying example: test_ints_are_commutative(
    x=0, y=1,
)
```

The function name indicates that we wanted to test commutativity, but the code indicates that it is the commutativity of subtraction - which obviously should fail!
Hypothesis has found the simplest possible counterexample.
Trying out the same with `+` instead of `-`, no counterexamples are found.

Now, we are interested in more subtle properties, having to do with our own functions rather than `+` or `-` of integers.
But first we must look a bit closer to the simple example code, since it uses a Python construct that we have not seen before: a **decorator**

#### 7.5.3. Decorators

A decorator in Python is a "function returning another function", as explained in:
<https://docs.python.org/3/glossary.html#term-decorator>

The decorator function, prefixed with the `@` symbol, is written on the line before the definition of the decorated function.
Decorators are **syntactic sugar**, which means that they can always be converted to "normal" syntax and thereby avoided:

```python
@dec
def f(...):
    ...
```

converts to:

```python
def f(...):
    ...
f = dec(f)
```

The decorator syntax is used when it is considered more readable, which is often the case in libraries.
Hypothesis is the first example we see at this course.
In the example above, converting the decorator to normal syntax would give us

```python
def test_ints_are_commutative(x, y):
    assert x - y == y - x

test_ints_are_commutative = given(st.integers(), st.integers())(test_ints_are_commutative)
```

If you are used to the decorator syntax, it will certainly look more readable!
However, if you have not seen it before, it may look like magic.

The strategies in Hypothesis work as random generators, and one can save their results by the `example()` method.
Thus one could even avoid the use of `given()` by storing the generated objects in a list:

```python
ints = []
for i in range(10):
    ints.append(st.integers().example())
```

You can run test functions by looping over such lists.
However, you will then miss some important functionalities that the `given()` function provides, so this is not the recommended way of using Hypothesis.
But trying it out in this way can help remove the magic.

#### 7.5.4. Strategies for graphs

Random values are generated by `strategies`, which have methods for many datatypes of Python, as explained in: <https://hypothesis.readthedocs.io/en/latest/data.html>

Here is a simple way to create a strategy for graphs:

```python
# generate small integers, 0...10
smallints = st.integers(min_value=0, max_value=10)

# generate pairs of small integers
twoints = st.tuples(smallints, smallints)

# generate lists of pairs of small integers
# where x != y for each pair (x, y)
st_edge_list = st.lists(twoints, unique_by=(lambda x: x[0], lambda x: x[1]))
```

The generation would also work without the `min`, `max`, and `unique` arguments, but it would produce more uninteresting example, for instance, very large integers as vertices.

As an example, let us test whether depth_first and breadth_first search always give the same results:

```python
@given(st_edge_list)
def test_searches(eds):
    G = Graph()
    for (a,b) in eds:
        G.add_edge(a, b)
    root = eds[0][0]
    assert breadth_first(G, root) == depth_first(G, root)
```

As expected, this is not the case:

```plain
Falsifying example: test_searches(
    eds=[(0, 2), (1, 0), (2, 1)],
)
```

The actual results can be seen by adding suitable print statements:

```plain
BREADTH FIRST [0, 2, 1]
DEPTH FIRST [0, 1, 2]
```

However, what is actually more interesting to test is if the two search methods find the same **sets** of nodes, even if the order differs.
Changing the assertion and testing again should not show any counterexamples:

```python
assert set(breadth_first(G, root)) == set(depth_first(G, root))
```

The `hypothesis` library provides many more ways to define, combine, and restrict strategies, but you can get started with these simple ones to generate random graphs.

### 7.5.5. Testing further graphs: TODO 
Example: Edge symmetry 