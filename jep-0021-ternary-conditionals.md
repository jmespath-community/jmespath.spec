# Ternary Conditional Expression

|||
|---|---
| **JEP**    | JEP-21
| **Author** | Maxime Labelle
| **Status** | accepted
| **Created**| 13-05-2025

## Abstract

This JEP introduces a new `expression` to support an _it-then-else_ ternary operation.

## Motivation

Most languages do have a ternary operator that is akin to an _if-then-else_ syntax as a single statement.
For instance, many languages do have the following syntax:

> \<condition\> ? \<true-result\> : \<false-result\>

While JMESPath does not have a ternary operator, the [closest approximation](https://github.com/jmespath-community/jmespath.spec/wiki/ternary-operator) using an expression like `` ( x || y ) && z `` unfortunately suffers from a limitation that makes it not really a true ternary operator. Specifically, if both `x` and `y` are false, the expression still falls back to evaluating `z`.

More complicated, "object-wrapping" expressions are possible such as `` (x && [y] || [z])[0] ``. Unfortunately, these appear to be more like a trick and are not user-friendly, especially for newcomers to the language.

For those reasons, we propose introducing a _true_ ternary operator to the language.

## Grammar Changes

The following updates to the grammar support a ternary operation:

```abnf
expression =/ ternary-expression

ternary-expression = expression "?" expression ":" expression
```
A `ternary-expression` (short for ternary conditional expression) is an expression taking the following form:

`` condition ? true-expression : false-expression ``

Where `condition`, `true-expression` and `false-expression` are JMESPath expressions.
A ternary conditional expression will evaluate to either its "true" expression or its "false" expression.
If the evaluation of the condition is not false, then the "true" expression is evaluated and used as the return value.
Otherwise, the "false" epression is evaluated and used as the return value.

The following values are considered to be false values in JMESPath:

- The boolean `false` JSON value
- The `null` JSON value
- An empty `""` JSON string
- An empty `{}` JSON objects
- An empty `[]` JSON array

Expressions within a ternary conditional expression are evaluated using a right-associative reading.
This means that a chain of ternary expression can be expressed without parentheses:

> a ? b :
> c ? d :
> e

## Precedence

The ternary condition expression is a new infix operator that has a higher precedence than the `|` pipe expression and lower precedence than the `||` and `&&` logical expressions. Thus the precedence of various expressions goes like this, from lowest to highest:

- `|` pipe expression
- `… ? … : …` ternary conditional expression
- `||` OR logical expression
- `&&` AND logical expression

## Compliance Tests

A new `ternary.json` test file will be added to the suite of compliance tests.

```json
[
	{
		"given": {
			"true": true, "false": false,
			"foo": "foo",  "bar": "bar", "baz": "baz", "qux": "qux", "quux": "quux"
		},
		"cases": [
			{ "expression": "true ? foo : bar", "result": "foo" },
			{ "expression": "false ? foo : bar", "result": "bar" },
			{ "expression": "`null` ? foo : bar", "result": "bar" },
			{ "expression": "`[]` ? foo : bar", "result": "bar" },
			{ "expression": "`{}` ? foo : bar", "result": "bar" },
			{ "expression": "'' ? foo : bar", "result": "bar" },
			
                        { "expression": "false ? foo | bar | @ : baz", "result": "baz" },
			{ "expression": "foo ? bar ? baz : qux : quux", "result": "baz" },

			{ "expression": "true || false ? foo : bar", "result": "foo" },
			{ "expression": "true && false ? foo : bar", "result": "bar" }
		]
	}
]
```

## Alternatives

Although achieving the same result is possible within the current specification, the "array-wrapping" trick referred to in the motivation section is not user-friendly.

Other alternatives considered introducing a more general "switch-case-when-else" syntax but we feel that a ternary operator is so ubiquitous in most languages that it is worth introducing in the language. This does not preclude revisiting a more general syntax in the future.

Finally, we considered [adding this feature as a function](https://github.com/springcomp/JmesPath.Net.Functions/wiki/iff). Although this can be done today in most implementations that provide an extension point, we think a ternary operator is the most user-friendly least surprising way to include this feature.
