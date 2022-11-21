# String Slices

|||
|---|---
| **JEP**    |  15
| **Author** | Maxime Labelle
| **Created**| 24-July-2022
| **SemVer** | MINOR
| **Status**| accepted

## Abstract

The original [JEP-5](https://github.com/jmespath-community/jmespath.spec/blob/main/jep-005-array-slices.md) introduced `slice-expression` in the grammar to slice specific portions of an array. While the syntax was specifically designed to operate on arrays, the syntactic grammar allows it after any expression.

This JEP introduces changes to allow `slice-expression` to operate on string types and act like a more powerful `substring()` function.

## Motivation

String manipulation functions are a frequently requested feature to be added to JMESPath. While introducing a whole host of string manipulation functions will certainly be proposed at some point, slicing strings is an easy extension to JMESPath that does not require any grammar change and is fully backwards compatible.

## Slices

_This section outline word changes to the _Slices_ documentation of the grammar **in bold**._

```abnf
slice-expression = [number] ":" [number] [ ":" [number] ]
```
A slice expression allows you to select a subset of an array **or string**. A slice has a `start`, `stop`, and `step` value. The general form of a slice is `[start:stop:step]`, but each component is optional and can be omitted.

> Slices in JMESPath have the same semantics as python slices. If you're familiar with python slices, you're familiar with JMESPath slices.

Given a `start`, `stop`, and `step` value, the sub elements in an array **or characters in a string** are extracted as follows:

- The first element in the extracted array **or first character in the extracted string** is the index denoted by `start`.
- The last element in the extracted array **or last character in the extracted string** is the index denoted by `end - 1`.
- The `step` value determines how many indices to skip after each element is selected from the array **or each character is selected from the string**.
   The default `step` value of `1` will not skip any indices and will return a contiguous subset of the original array **or a substring of the original string**.
   A `step` value greater than `1` will skip indices while extracting elements from an array or **characters from a string**. For instance, a `step` value of `2` will skip every other element **or character**.
   Negative `step` values start from the end of the array **or string** and extract elements **or characters** in reverse order.

Slice expressions adhere to the following rules:

- If a negative `start` position is given, it is calculated as the total length of the array **or string** plus the given start position.
- If no `start` position is given, it is assumed to be `0` if the given `step` is greater than `0` or the end of the array **or string** if the given `step` is less than `0`.
- If a negative `stop` position is given, it is calculated as the total length of the array **or string** plus the given `stop` position.
- If no `stop` position is given, it is assumed to be the length of the array **or string** if the given `step` is greater than `0` or `0` if the given `step` is less than `0`.
- If the given `step` is omitted, it it assumed to be `1`.
- If the given `step` is `0`, an error MUST be raised.
- If the **object** being sliced is not an array **or string**, the result is `null`.
- If the **object** being sliced is an array **or string** and yields no results, the result MUST be an empty array **or empty string**.

### Examples

Slicing operates on a strings exactly as if a string were thought of as an array of characters.

- `` search( foo[0:4], {"foo": "hello, world!" } -> "hell" ``
- `` search( [::], 'raw-string') -> "raw-string" ``
- `` search( [::2], 'raw-string') -> "rwsrn" ``
- `` search( [::-1], 'raw-string') -> "gnirts-war" ``

## Compliance tests

A new `string_slices.json` file will be added to the compliance test suite.

## History

N/A
