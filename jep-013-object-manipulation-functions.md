# Object Manipulation Functions

|||
|---|---
| **JEP**    |  13
| **Author** | Jonathan Stewmon
| **Created**| 21-March-2016
| **SemVer** | MINOR
| **Status**| accepted

## Abstract

As a JSON querying language, JMESPath has long needed some functions to manipulate JSON objects.

This JEP introduces built-in functions to extract a list of key/value pairs from a JSON object and conversely converting an array of key/value pairs to a JSON object.

## Specification

### items

```
array[array[any]] items(object $obj)
```

Returns an array of key value pairs for the provided object `$obj`. Each pair is a 2-item array with the first item being the key and the second item being the value. This function is the inverse of the `from_items()` function.

Note that because JSON hashes are inheritently unordered, the key value pairs of the provided object `$obj` are inheritently unordered.  Implementations are not required to return values in any specific order. 

For example, given the input:

```json
{"a": "first", "b": "second", "c": "third"}
```

The expression ``items(@)`` could have any of these return values:

- ``[["a", "first"], ["b", "second"], ["c", "third"]]``
- ``[["a", "first"], ["c", "third"], ["b", "second"]]``
- ``[["b", "second"], ["a", "first"], ["c", "third"]]``
- ``[["b", "second"], ["c", "third"], ["a", "first"]]``
- ``[["c", "third"], ["a", "first"], ["b", "second"]]``
- ``[["c", "third"], ["b", "second"], ["a", "first"]]``

If you would like a specific order, consider using the ``sort_by`` function.

### Examples

|Given|Expression|Result
|---|---|---
|``{"a": "first", "b": "second"}``|``items(@)``|``[["b", "second"], ["a", "first"]]``
|``{"z": "last", "b": "second"}``|``sort_by(items(@), &[0])``|``[["b", "second"], ["z", "last"]]``
|``{"z": "last", "b": "second"}``|``sort_by(items(@), &[1])``|``[["z", "last"], ["b", "second"]]``

### from_items

```
object from_items(array[array[any]] $arg)
```
Returns an object from the provided array of key value pairs. This function is the inverse of the `items()`.

### Examples

|Given|Expression|Result
|---|---|---
|``[["one", 1], ["two", 2]]``|``from_items(@)``|``{"one": 1, "two": 2}``
|``[["one", 1], ["two", 2], ["one", 3]]``|``from_items(@)``|``{"one": 3, "two": 2}``

### zip

```
array[array[any]] zip([array[any] $arg, [, array[any] $...]])
```

Accepts one or more arrays as arguments and returns an array of arrays in which the *i-th* array contains the *i-th* element from each of the argument arrays. The returned array is truncated to the length of the shortest argument array.

### Examples

|Expression|Result
|---|---
|``zip(`["a", "b"]`, `[1, 2]`)``|``[["a", 1], ["b", 2]]``
|``zip(`["a", "b", "c"]`, `[1, 2]`)``|``[["a", 1], ["b", 2]]``

## Compliance tests

A new `objects.json` file will be added to the compliance test suite.
