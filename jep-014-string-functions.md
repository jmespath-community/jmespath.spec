# String Functions

|||
|---|---
| **JEP**    |  14
| **Author** | Maxime Labelle, Chris Armstrong (GorillaStack), Richard Gibson
| **Created**| 13-October-2022
| **SemVer** | MINOR
| **Status**| Draft

## Abstract

This JEP introduces a core set of useful string manipulation functions. Those functions are modeled from functions found in popular programming languages such as JavaScript and Python.

## Specification

Some string manipulation functions bring the new concept of _optional arguments_ to JMESPath functions. The specification paragraph on function evaluation must thus be changed accordingly – highlighted in **bold** in the text below:

_Functions can ~~either~~ have a specific arity, **a range of valid – minimum and maximum – number of arguments** or be variadic with a minimum number of arguments. If a function-expression is encountered where the arity does not match or the minimum number of arguments for a variadic function is not provided, then implementations must indicate to the caller that an invalid-arity error occurred. How and when this error is raised is implementation specific._



### find_first

```
int find_first(string $subject, string $sub[, int $start[, int $end]])
```
Given the `$subject` string, `find_first()` returns the zero-based index of the first occurence where the `$sub` substring appears in `$subject` or `null` if it does not appear.

The `$start` and `$end` parameters are optional and allow restricting the range within `$subject` in which `$sub` must be found.

- If `$start` is omitted, it defaults to `0` (which is the start of the `$subject` string).
- If `$end` is omitted, it defaults to `length(subject)` (which is past the end of the `$subject` string).

Contrary to similar functions found in most popular programming languages, the `find_first()` function does not return `-1` if no occurrence of the substring can be found. Instead, it returns `null` for consistency reasons with how JMESPath behaves.

### Examples

| Given | Expression | Result
|---|---|---
| `"subject string"` | `` find_first(@, 'string') `` |  `8`
| `"subject string"` | `` find_first(@, 'string', `0`) `` |  `8`
| `"subject string"` | `` find_first(@, 'string', `0`, `14`) `` |  `8`
| `"subject string"` | `` find_first(@, 'string', `-99`, `100`) `` |  `8`
| `"subject string"` | `` find_first(@, 'string', `0`, `13`) `` |  `null`
| `"subject string"` | `` find_first(@, 'string', `8`) `` |  `8`
| `"subject string"` | `` find_first(@, 'string', `9`) `` |  `null`
| `"subject string"` | `` find_first(@, 's') `` |  `0`
| `"subject string"` | `` find_first(@, 's', 1) `` |  `8`
| `"subject string"` | `` find_first(@, '') `` |  `null`

### find_last

```
int find_last(string $subject, string $sub[, int $pos])
```
Given the `$subject` string, `find_last()` returns the zero-based index of the last occurence where the `$sub` substring appears in `$subject` or `null` if it does not appear.

The `$pos` parameter is optional and allows restricting the maximum index within `$subject` at which to search for `$sub`.
If this is parameter omitted, it defaults to `length(subject) - 1` (which is is the end of the `$subject` string).

Contrary to similar functions found in most popular programming languages, the `find_last()` function does not return `-1` if no occurrence of the substring can be found. Instead, it returns `null` for consistency reasons with how JMESPath behaves.

### Examples

| Given | Expression | Result
|---|---|---
| `"subject string"` | `` find_last(@, 'string') `` |  `8`
| `"subject string"` | `` find_last(@, 'string', 8) `` |  `8`
| `"subject string"` | `` find_last(@, 'string', 7) `` |  `null`
| `"subject string"` | `` find_last(@, 's', 8) `` |  `8`
| `"subject string"` | `` find_last(@, 's', 7) `` |  `0`

### lower

```
string lower(string $subject)
```
Returns the lowercase `$subject` string using Unicode default casing conversion specification.

### Examples

| Given | Expression | Result
|---|---|---
| `"STRING"` | `` lower(@) `` |  `"string"`

### pad_left

```
string pad_left(string $subject, number $width[, string $pad])
```

Given the `$subject` string, `pad_left()` adds characters to the beginning and returns a string of length at least `$width`.

The `$pad` optional string parameter specifies the padding character.
If omitted, it defaults to an ASCII space (U+0020).
If present, it MUST have length 1, otherwise an error MUST be raised.

If the `$subject` string has length greater than or equal to `$width`, it is returned unmodified.

### Examples

| Given | Expression | Result
|---|---|---
| `"string"` | `` pad_left(@, `0`) `` |  `"string"`
| `"string"` | `` pad_left(@, `5`) `` |  `"string"`
| `"string"` | `` pad_left(@, `10`) `` |  `"    string"`
| `"string"` | `` pad_left(@, `10`, '-') `` |  `"----string"`

### pad_right

```
string pad_right(string $subject, number $width[, string $pad])
```

Given the `$subject` string, `pad_right()` adds characters to the end and returns a string of length at least `$width`.

The `$pad` optional string parameter specifies the padding character.
If omitted, it defaults to an ASCII space (U+0020).
If present, it MUST have length 1, otherwise an error MUST be raised.

If the `$subject` string has length greater than or equal to `$width`, it is returned unmodified.

### Examples

| Given | Expression | Result
|---|---|---
| `"string"` | `` pad_right(@, `0`) `` |  `"string"`
| `"string"` | `` pad_right(@, `5`) `` |  `"string"`
| `"string"` | `` pad_right(@, `10`) `` |  `"string    "`
| `"string"` | `` pad_right(@, `10`, '-') `` |  `"string----"`

### replace

```
string replace(string $subject, string $old, string $new[, number $count])
```
Given the `$subject` string, `replace()` replaces occurrences of the `$old` substring with the `$new` substring.

The `$count` optional integer specifies how many occurrences of the `$old` substring in `$subject` are replaced. If this parameter is omitted, all occurrences are replaced. If `$count` is not an integer or is negative, an error MUST be raised.

The `replace()` function has no effect if `$count` is `0`.

### Examples

| Given | Expression | Result
|---|---|---
| `"aabaaabaaaab"` | `` replace(@, 'aa', '-', `0`) `` |  `"aabaaabaaaab"`
| `"aabaaabaaaab"` | `` replace(@, 'aa', '-', `1`) `` |  `"-baaabaaaab"`
| `"aabaaabaaaab"` | `` replace(@, 'aa', '-', `2`) `` |  `"-b-abaaaab"`
| `"aabaaabaaaab"` | `` replace(@, 'aa', '-', `3`) `` |  `"-b-ab-aab"`
| `"aabaaabaaaab"` | `` replace(@, 'aa', '-') `` |  `"-b-ab--b"`

### split

```
array[string] split(string $subject, string $search[, number $count])
```

Given the `$subject` string, `split()` breaks on ocurrences of the string `$search` and returns an array.

The `split()` function returns an array containing each partial string between occurrences of `$search`. If  `$subject` contains no occurrences of the `$search` string, an array containing just the original `$subject` string will be returned.

If the `$search` argument is an empty string, `split()` breaks on every character and returns an array containing each character from the `$subject` string. Thus, if `$subject` is _also_ an empty string, `split()` returns an empty array.

The `$count` optional integer specifies the maximum number of split points within the `$search` string.
If this parameter is omitted, all occurrences are split. If `$count` is not an integer or is negative, an error MUST be raised.

If `$count` is equal to `0`, `split()` returns an array containing a single element, the `$subject` string.

Otherwise, the `split()` function breaks on occurrences of the `$search` string up to `$count` times. The last string in the resulting array containing the remaining contents of `$subject` unmodified.

**Note**: The `split()` function was [originally designed by Chris Armstrong](https://github.com/GorillaStack/jmespath.site/blob/master/docs/proposals/string-manipulation.rst). However, its behaviour has been slightly altered for consistency reasons.

### Examples

| Expression | Result
|---|---
| `split('', '')` | `[]`
| `split('all chars', '')` | `[ "a", "l", "l", " ", "c", "h", "a", "r", "s" ]`
| `split('/', '/')` | `[ "", "" ]` |
|`` split('average\|-\|min\|-\|max\|-\|mean\|-\|median', '\|-\|') `` | `[ "average", "min", "max", "mean", "median" ]` 
|`` split('average\|-\|min\|-\|max\|-\|mean\|-\|median', '\|-\|', `3`) `` | `[ "average", "min", "max", "mean\|-\|median" ]`
|`` split('average\|-\|min\|-\|max\|-\|mean\|-\|median', '\|-\|', `2`) `` | `[ "average", "min", "max\|-\|mean\|-\|median" ]`
|`` split('average\|-\|min\|-\|max\|-\|mean\|-\|median', '\|-\|', `1`) `` | `[ "average", "min\|-\|max\|-\|mean\|-\|median" ]`
|`` split('average\|-\|min\|-\|max\|-\|mean\|-\|median', '\|-\|', `0`) `` | `[ "average\|-\|min\|-\|max\|-\|mean\|-\|median" ]`
| `split('average\|-\|min\|-\|max\|-\|mean\|-\|median', '-')` | `[ "average\|", "\|min\|", "\|max\|", "\|mean\|", "\|median" ]`

## Specification

### trim

```
string trim(string $subject[, string $chars])
```
Given the `$subject` string, `trim()` removes the leading and trailing characters found in `$chars`.

The `$chars` optional string parameter represents a set of characters to be removed. If this parameter is not specified, or is an empty string, whitespace characters are removed from the `$subject` string. Whitespaces are defined by the Unicode standard as codepoints having the `White_Space` property set to `Yes`.

### Examples

| Given | Expression | Result
|---|---|---
| `"   subject string   "` | `` trim(@) `` |  `"subject string"`
| `"   subject string   "` | `` trim(@, '') `` |  `"subject string"`
| `"   subject string   "` | `` trim(@, ' ') `` |  `"subject string"`
| `"   subject string   "` | `` trim(@, 's') `` |  `"   subject string   "`
| `"   subject string   "` | `` trim(@, 'su') `` |  `"   subject string   "`
| `"   subject string   "` | `` trim(@, 'su ') `` |  `"bject string"`
| `"   subject string   "` | `` trim(@, 'gsu ') `` |  `"bject strin"`

### trim_left

```
string trim_left(string $subject[, string $chars])
```
Given the `$subject` string, `trim_left()` removes the leading characters found in `$chars`.

Like for the `trim()` function, the `$chars` optional string parameter represents a set of characters to be removed. `trim_left()` defaults to removing whitespace characters if `$chars` is not specified or is an empty string.

### Examples

| Given | Expression | Result
|---|---|---
| `"   subject string   "` | `` trim_left(@) `` |  `"subject string   "`
| `"   subject string   "` | `` trim_left(@, 's') `` |  `"   subject string   "`
| `"   subject string   "` | `` trim_left(@, 'su') `` |  `"   subject string   "`
| `"   subject string   "` | `` trim_left(@, 'su ') `` |  `"bject string   "`
| `"   subject string   "` | `` trim_left(@, 'gsu ') `` |  `"bject string   "`

### trim_right

```
string trim_right(string $subject[, string $chars])
```
Given the `$subject` string, `trim_right()` removes the trailing characters found in `$chars`.

Like for the `trim()` and `trim_left()` functions, the `$chars` optional string parameter represents a set of characters to be removed. `trim_right()` defaults to removing whitespace characters if `$chars` is not specified or is an empty string.

### Examples

| Given | Expression | Result
|---|---|---
| `"   subject string   "` | `` trim_right(@) `` |  `"   subject string"`
| `"   subject string   "` | `` trim_right(@, 's') `` |  `"   subject string   "`
| `"   subject string   "` | `` trim_right(@, 'su') `` |  `"   subject string   "`
| `"   subject string   "` | `` trim_right(@, 'su ') `` |  `"   subject string"`
| `"   subject string   "` | `` trim_right(@, 'gsu ') `` |  `"   subject strin"`

### upper

```
string upper(string $subject)
```
Returns the uppercase `$subject` string using Unicode default casing conversion specification.

| Given | Expression | Result
|---|---|---
| `"string"` | `` upper(@) `` |  `"STRING"`

## Compliance

A new `string_functions.json` file will be added to the compliance tests.
