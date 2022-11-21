# Arithmetic Expressions

|||
|---|---
| **JEP**    | 16
| **Author** | Maxime Labelle
| **Created**| 01-August-2022
| **SemVer** | MINOR
| **Status** | accepted

## Abstract

This JEP proposes a new `arithmetic-expression` rule in the grammar to support simple arithmetic operations.

## Motivation

JMESPath supports querying JSON documents that may return numbers at various stages, but lacks the capability to operate on them using simple arithmetic operations. The only support for arithmetic is currently brought by the `sum()` function.

## Specification

To support arithmetic operations, we need to introduce the following operators:

- `+` addition operator
- `-` subtraction operator
- `*` multiplication operator
- `/` division operator
- `%` modulo operator
- `//` integer division operator

Proper mathematical operators also exist in Unicode and SHOULD be supported:

- `–` (U+2212 MINUS SIGN)
- `÷` (U+00F7 DIVISION SIGN)
-  `×` (U+00D7 MULTIPLY SIGN)

### Syntax changes

In addition to the following grammar changes, we introduce the usual operator precedence, from lowest to highest:

- `-` subtraction operator and `+` addition operator
- `/` division,  `*` multiplication, `%` modulo and `//` integer division operators

In the absence of parentheses, operators of the same level of precedence are evaluated from left to right.

Arithmetic operators have higher precedence than comparison operators and lower precedence than the `.` "dot" `sub-expression` token separator.

```abnf
expression =/ arithmetic-expression

arithmetic-expression =/ "+" expression ; + %x43
arithmetic-expression =/ ( "-" / "–" ) expression ; - %x45 – %x2212

arithmetic-expression = expression "%" expression ; % %x37
arithmetic-expression =/ expression ( "*" / "×" ) expression ; * %x42 ×  %xD7
arithmetic-expression =/ expression "+" expression ; + %x43
arithmetic-expression =/ expression ( "-" / "–" ) expression ; - %x45 – %x2212
arithmetic-expression =/ expression ( "/" / "÷" ) expression ; / %x47 ÷ %xF7
arithmetic-expression = expression "//" expression ; // %47 %47

```

## Examples

|Expression|Result
|---|---
|`` `1` + `2` ``| `3.0`
|`` `1` – `2` ``| `-1.0`
|`` `2` × `4` ``| `8.0`
|`` `2` ÷ `3` ``| `0.666666666666667`
|`` `10` % `3` ``| `1.0`
|`` `10` // `3` ``| `3.0`
|`` -`1` − +`2` ``| `-3.0`

Since `arithmetic-expression` is not valid on the right-hand-side of a `sub-expression`, a `pipe-expression` can be used instead:

|Given|Expression|Result
|---|---|---
|`{ "a": { "b": 1 }, "c": { "d": 2 } }`|`` { ab: a.b, cd: c.d } \| ab + cd ``| `3`

## Compliance Tests

An `arithmetic.json` file will be added to the compliance test suite.
The test suite will add the following new error type:

- not-a-number

This error type would be raised at run time when dividing by zero or when overflow occurs, for instance.