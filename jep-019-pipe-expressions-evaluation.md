# Evaluation of Pipe Expressions

|||
|---|---
| **JEP**    |  19
| **Author** | Maxime Labelle
| **Created**| 29-October-2022
| **SemVer** | MINOR
| **Status**| accepted

## Abstract

This JEP introduces changes that outline the exact intended behaviour of evaluating a `pipe-expression`. It also clarifies an ambiguity when evaluating a `sub-expression` where the left-hand-side evaluates to `null`.

## Motivation

### Sub Expression Evaluation

The specification for [`sub-expression`](https://jmespath.org/specification.html#subexpressions) outlines how it should be evaluated using pseudocode:

```
left-evaluation = search(left-expression, original-json-document)
result = search(right-expression, left-evaluation)
```

This is slightly ambiguous, as many compliance tests expect the result to be `null` when the left-hand-side evaluates to  `null`. So, the real pseudocode shoud in fact be:

```
left-evaluation = search(left-expression, original-json-document)
if left-evaluation is `null` then result = `null`
else result = search(right-expression, left-evaluation)
```

### Pipe Expression Evaluation

In contrast, however, it seems intuitive for `pipe-expression` to evaluate as was originally outlined by the first pseudocode fragment referred to a above.

```
left-evaluation = search(left-expression, original-json-document)
result = search(right-expression, left-evaluation)
```

Which means that the evaluation should still happens if the left-hand-side is `null`.

## Specification

### Sub Expressions

The paragraph on sub expressions in the specification will be updated as follows, with changes outlined in **bold**:

_A subexpression is evaluted as follows:_

- _Evaluate the expression on the left with the original JSON document._
- _**If the result of the left expression evaluation is `null`, return `null`.**_
- _**Otherwise, e**~~E~~valuate the expression on the right with the result of the left expression evaluation._

_In pseudocode:_

```
left-evaluation = search(left-expression, original-json-document)
If result is `null` the result = `null`
else result = search(right-expression, left-evaluation)
```

### Pipe Expressions

Likewise the paragraph on pipe expressions in the specification will be updated as follows:

_A pipe expression combines two expressions, separated by the | character. It is similar to a sub-expression with ~~two~~**the following** important distinctions:_

1. _Any expression can be used on the right hand side. A `sub-expression` restricts the type of expression that can be used on the right hand side._
2. _A `pipe-expression` stops projections on the left hand side ~~for~~ **from** propagating to the right hand side. If the left expression creates a projection, it does not apply to the right hand side._
3. _**A `pipe-expression` evaluates the right expression unconditionally.  A `sub-expression` shortcuts evaluation if the result of the left expression evaluation is `null`**_

_**In pseudocode:**_

```
left-evaluation = search(left-expression, original-json-document)
result = search(right-expression, left-evaluation)
```

### Example

| Expression | Result
|---|----
| `` `null` . [@] `` | `null`
| `` `null` `| [@] `` | `[ null ]`

## Compliance

The `pipe.json` compliance test file will be updated with the example from this JEP.
