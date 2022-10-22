# String Functions

|||
|---|---
| **JEP**    |  14
| **Author** | Maxime Labelle, Chris Armstrong (GorillaStack)
| **Created**| 13-October-2022
| **SemVer** | MINOR
| **Status**| Draft

## Abstract

This JEP introduces a core set of useful string manipulation functions. Those functions are modeled from functions found in popular programming languages such as JavaScript and Python.

## Specification

### find_first

```
int find_first(string $subject, string $sub[, int $start[, int $end]])
```
Given the `$subject` string, `find_first()` returns the index of the first occurence where the `$sub` substring appears in `$subject` or `null`.

The `$start` and `$end` parameters are optional and allow to select where `find_first()` must perform its search within `$subject`.

- If `$start` is omitted, it defaults to `0` which is the start of the `$subject` string.
- If `$end` is omitted, it defaults to `length($subject) -1` which is the end of the string.

Contrary to similar functions found in most popular programming languages, the `find_first()` function does not return `-1` if the occurrence of the `$sub` substring cannot be found. Instead, it returns `null` for consistency reasons with how JMESPath behaves. 

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

### find_last

```
int find_last(string $subject, string $sub[, int $pos])
```
Given the `$subject` string, `find_last()` returns the index of the last occurence where the `$sub` substring appears in `$subject` or `null`.

The `$pos` optional parameter allow to select where `find_last()` must perform its search within `$subject`.

If this parameter is omitted, It defaults to `length($subject) - 1` which is the end of the `$subject` string. `find_last()` then searches backwards in the `$subject` string for the first occurrence of `$sub`.

Likewise, the `find_last()` function does not return `-1` if the occurrence of the `$sub` substring cannot be found. Instead, it returns `null` for consistency reasons with how JMESPath behaves. 

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
Returns the lowercase `$subject` string.

### Examples

| Given | Expression | Result
|---|---|---
| `"STRING"` | `` lower(@) `` |  `"string"`

### pad_left

```
string pad_left(string $subject, number $width[, string $pad])
```

Given the `$subject` string, `pad_left()` adds `$width - length($subject)` characters to the beginning of the string and returns a string of length `$width`.

The `$pad` optional string parameter specifies the padding character to use. If this parameter is omitted, `pad_left()` adds one or more ASCII space characters to the beginning of the string. If present, the `$pad` string must have a single character, otherwise, an error MUST be raised.

The `pad_left()` function has no effect if `$width` is less than, or equal to the length of the ` $subject` string.

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

Given the `$subject` string, `pad_right()` adds `$width - length($subject)` characters to the end of the string and returns a string of length `$width`.

The `$pad` optional string parameter specifies the padding character to use. If this parameter is omitted, `pad_right()` adds ASCII space characters to the end of the string. If present, the `$pad` string must have a single character, otherwise, an error MUST be raised.

The `pad_right()` function has no effect if `$width` is less than, or equal to the length of the ` $subject` string.

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
Given the `$subject` string, `replace()` replaces all occurrences of the `$old` substring with the `$new` substring.

The `$count` optional parameter specifies how many occurrences of the `$old` substring in `$subject` are replaced. If this parameter is omitted, all occurrences are replaced. If `$count` is negative, an error MUST be raised.

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

Given string `$subject`, `split()` breaks on occurrences of the string `$search` and returns an array.

The `split()` function returns an array containing each partial string between occurrences of `$search`. If  `$subject` contains no occurrences of the `$search` string, an array containing just the original `$subject` string will be returned.

The `$count` optional parameter specifies how many occurrences of the `$search` string are split. If this parameter is omitted, all occurrences are split. If `$count` is negative, and error MUST be raised.

If `$count` is equal to `0`, `split()` returns an array containing a single element, the `$subject` string.

Otherwise, the `split()` function breaks on occurrences of the `$search` string up to `$count` times. The last string in the resulting array containing the remaining contents of `$subject` unmodified.

**Note**: The `split()` function was [originally designed by Chris Armstrong](https://github.com/GorillaStack/jmespath.site/blob/master/docs/proposals/string-manipulation.rst). However, its behaviour has been slightly altered for consistency reasons.

### Examples

| Expression | Result
|---|---
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

The `$chars` optional string parameter represents a set of characters to be removed. If this parameter is not specified, or is an empty string, whitespace characters are removed from the `$subject` string.

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
Returns the uppercase `$subject` string.

| Given | Expression | Result
|---|---|---
| `"string"` | `` upper(@) `` |  `"STRING"`

## Compliance

A new `string_functions.json` file will be added to the compliance tests.
