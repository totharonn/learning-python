# How do variables work in Python?

Python's behaviour is odd sometimes.  
Let's take a closer look on this function which looks quite innocent:
```python3
def twice(something):
    something += something
    return something
```

For numbers and strings it works as supposed to work (`+` means concatenation for strings[^1])
```python3
a = 2
print(twice(a), a)
```
shall output `4 2` and similarly
```python3
a = "two"
print(twice(a), a)
```
shall output `twotwo two`. It's worth two mention though the value of 'twice two' depends on the type of two which essentially hints to the interpreter how to understand it. In the first case `a` refers to a number, so its twice supposed to be a number, and twiceness is meant to be in the usual numerical way. In the second case `"two"` is meant as a string, roughly spoken a written word, its twice is supposed to be the word written twice. One might mention that there is a chance python interpreter doesn't know what "two" means. Indeed, so let's consider the following excercise. What is the output of the following snippet?
```python3
a = "2"
print(twice(a), a)
```

## The `bool` type

The first surprise shows up with booleans.
```python3
a = True
print(twice(a), a)
```
shall output `2 True`. Why? Python happened to choose implementing `bool` as a subtype of `int`[^2] and joined to the convention to represent `True` as the int 1 and `False` as the int 0[^3]. So practically, in Python `True` is just an alias of `1`. The multiplication (`*`) operator works on `0` and `1` exactly the same way as the `and` operator works on `True` and `False`. Nevertheless `and` and `*` aren't the same. What's the difference between them?

As we've seen recently, `bool` is embedded into the type `int`. There is a reversed connection between these types as well. Each integer -- and each _object_ in Python -- has an assigned truth value.
```python3
if 0:
   ... # never reached

While 1:
   ... # forever
```

An object can be _truthy_ or _falsy_ in Python, corresponding to its assigned truth value. Usually most objects are truthy, except[^4]
* `None` and `False`
* Numbers representing 0. (like `0` and `0.0`)
* Empty sequences and collections. (Emptiness of an list can be tested like below
  ```python3
  if a_list:
    ... # Do what you want to do with an empty list 
  ```
  )

## Mutables and immutables

One might have wondered why we tested `twice` not just with printing the value it returned, but the argument as well which we passed. It was because we wanted to know, whether it has got any _side effects_ or not. Instead of explaining what a side effect is let me show two further examples.
```python3
a = (1, 2, 3)
print(twice(a), a)
```
Nothing interesting here, it outputs `(1, 2, 3, 1, 2, 3) (1, 2, 3)`. Note `a` refers to an object of the type `tuple`.
```python3
a = [1, 2, 3]
print(twice(a), a)
```
It outputs `[1, 2, 3, 1, 2, 3] [1, 2, 3, 1, 2, 3]` and indeed, this is quite an illustration of what a side effect is. `twice` is supposed to return the double of the passed argument without changing the argument itself, but now it has changed the argument too. The most surprising is in this phenomenon that we made the same thing as right before we just passed a sequence of three numbers to `twice`. The only difference we've got that in the first time we've used parentheses an in the second time we've used brackets. First we have to note there is a difference between the meaning of `(1, 2, 3)` and `[1, 2, 3]`. The first refers to a `tuple` the latter to a `list`. Both `tuple` and `list` are for representing finite sequence of objects and both are random access[^5] and about equally fast. Why do we have two types for the same thing then? Well, as we've seen, there is a difference between them. `tuple` is somehow resisting against side effects and now we are going to understand the mechanism of its resistance and why Python chosen this strategy to avoid undesired side effects.

### What is a variable? `is` vs. `==`

In order to understand the concept of mutability in Python we have to understand what is a variable. Let's take a look at the code line below:
```python3
a = [1, 2, 3]
```
in a low-level language like C we would write something like this:
```C
int[3] a = {1,2,3};
```
which effects like it allocates a block of size three times the size of `int` in the stack and assigns its address to the variable `a` and just after this is done fills the allocated memory with the values on the right side.
Python assignment works in the opposite way[^6], it creates first the `list` object `[1, 2, 3]` and once it's done, the interpreter binds its value to the variable, which is here a reference of that object. (This is the way python achieves dynamic typing in, interpreter doesn't have to know, how much memory has to be allocated to the referred object, `a` the variable is just a reference of it) In the case the right side is a reference, the left side will bind to the referee, not the reference as we'll see in the example below.
```python3
a = [1, 2, 3]           # Creates the list, and binds it to a
b = a                   # binds b to the same list
a.append(4)             # the referred list is now [1, 2, 3, 4]
print(a is b)           # "True"
print(id(a), id(b))     # prints the same address twice
print(b)                # ?
a = [1, 2, 3]           # creates a new list and binds it to a
print(a is b)           # "False"
print(id(a), id(b))     # prints two different addresses
print(a, b)             # ?
```
The snippet above verifies that `b` isn't bound to `a`, but the object that `a` refers to. We used the `is` _keyword_ and the `id` _built-in_ function to get some insight. The latter returns the memory address of the argument, or the object referred by the argument, and `is` is a logical operator between two references, which returns True if and only, if the referree of both operand is the same that is `a is b` equivalent `id(a) == id(b)` where the operator `==` means the semantical equality.  
**Exercise** Which is the right: If `a is b` then `a == b` or if `a == b` then `a is b`?  
**Exercise** The following code prints `True False`
```python3
a = (1, 2, 3)
b = (1, 2, 3)
print(a==b, a is b)
```
but the following prints `True True`
```
a = 2
b = 2
print(a == b, a is b)
```
Can you explain why? (this phenomenon doesn't follow what I've written above, consider it like an experimental fact, and try to explain it)

### The difference between `list` and `tuple`

The root difference between the types `list` and `tuple` is that the first is _mutable_ and the latter is _immutable_ which means once it's created it is not able to be changed.


## Footnotes
[^1]: See: https://docs.python.org/3/library/stdtypes.html#common-sequence-operations 
 (string is a "text sequence" type)
[^2]: See: https://peps.python.org/pep-0285/ (PEP means Python Enhancement Proposal. PEPs are quite useful resources to give us some insight how the language developed.)
[^3]: This convention is pretty widespread but not exclusive at all; the POSIX `true` command exits with zero and `false` with a non-zero return value.  
See: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/true.html#
[^4]: See: https://docs.python.org/3/library/stdtypes.html#truth-value-testing
[^5]: See: https://en.wikipedia.org/wiki/Random_access
[^6]: See: https://docs.python.org/3/reference/simple_stmts.html#assignment-statements

