# String Manipulation (GorillaStack)

|||
|---|---
| **JEP**    | 16
| **Author** | Chris Armstrong (Gorilla Stack)
| **Status** | draft
| **Created**| 31-10-2019

## Abstract

This document proposes expanding the list of available string manipulation
functions in the spec.

## Motivation

The specification only contains basic string search and manipulation functions,
such as starts_with() and contains(), which are insufficient for other use
cases, such as replacing parts of strings, splitting strings, using regular expressions
for searching or manipulation of strings, or other common string manipulations
found in the common libraries of mainstream languages such as Python or JavaScript.

## Specification

The following functions will be added to the specification:

### split

```
array[string] split(string $subject, string $split_str, number $max = -1)
```

Given string `$subject`, split will split on occurrences string `$split_str`, up to and including
$max times (if specified).

The result is an array containing each partial string between occurrences of `$split_str`. If the
string contains no occurrences of  $split_str, an array containing just the original `$subject`
string will be returned.

`$max` is a optional parameter. When specified as an integer greater than zero, it will split the
string up to `($max - 1)` times, with the last string in the array containing the remaining
contents of `$subject` unmodified.

If not specified or it is less than zero, the string will be split on every occurrence of `$split_str`.

When `$max` is equal to zero, it will behave as if the string is not split at all, and return
an array containing one element, the original string.

### Examples

| Expression | Result 
|---|---
| `split('average\|-\|min\|-\|max\|-\|mean\|-\|mode\|-\|median', '\|-\|')` | `["average", "min", "max", "mean", "mode", "median"]`|
| `split('average\|-\|min\|-\|max\|-\|mean\|-\|mode\|-\|median', '\|-\|', 3)` | `["average", "min", "max\|-\|mean\|-\|mode\|-\|median"]`
| `split('average\|-\|min\|-\|max\|-\|mean\|-\|mode\|-\|median', '-')` | `["average\|", "\|min\|", "\|max\|", "\|mean\|", "\|mode\|", "\|median"]`
| `split('average\|-\|min\|-\|max\|-\|mean\|-\|mode\|-\|median', '\|-\|', 0)` | `["average\|-\|min\|-\|max\|-\|mean\|-\|mode\|-\|median"]`
| `split('every character', '')` | `["e", "v", "e", "r", "y", " ", "c", "h", "a", "r", "a", "c", "t", "e", "r"]`

## Impact

There should be no impact to existing users, as these are new functions.
