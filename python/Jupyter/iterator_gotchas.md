# Python Iterator Gotcha's
Python iterators are very useful for memory efficient list processing but they come with a few gotcha's.

## 1. Try Except

Consider the following:
``` python
foo = [0]

out = [1/i for i in foo]
```

The above code will result in a `ZeroDivisionError`.
We can catch this with the following code:

``` python
foo = [0]
try:
    out = [1/i for i in foo]
except Exception:
    print("We caught the error")
```

But suppose that the second statement is inside a generator:

``` python
foo = [0]
try:
    out = (1/i for i in foo)
except Exception:
    print("We caught the error")

bar = list(out)
```
The above code would still result in a `ZeroDivisionError` dispite the error visually occuring inside the try except block.
This occurs because the division by zero actually occurs in the line `bar = list(out)`.
