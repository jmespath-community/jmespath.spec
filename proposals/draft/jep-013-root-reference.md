# Root Reference

|||
|---|---
| **JEP**    | 13
| **Author** | Maxime Labelle
| **Status** | draft
| **Created**| 25-02-2022

## Abstract

This document proposes grammar modifications to JMESPath 
to support referring to the original input JSON document
inside an expression.

## Motivation

As a JMESPath expression is being evaluated, the current scope changes.
Given a simple sub expression such as `foo.bar`, first the `foo`
expression is evaluated with the starting input JSON document, and the
result of that expression is then used as the current scope when the
`bar` element is evaluated.

Once we’ve drilled down to a specific scope, there is no way, in the
context of the currently evaluated expression, to refer to any
elements outside of that element.

A common request when querying JSON objects is the ability to refer to
the input JSON document.

It is also a common occurrence to request access to the original input
JSON document original input JSON document.

For example, suppose we had this data:

```
{"first_choice": "WA",
 "states": [
   {"name": "WA", "cities": ["Seattle", "Bellevue", "Olympia"]},
   {"name": "CA", "cities": ["Los Angeles", "San Francisco"]},
   {"name": "NY", "cities": ["New York City", "Albany"]},
 ]
}
```

Let’s say we wanted to get the list of cities of the state corresponding
to our `first_choice` key.  We’ll make the assumption that the state
names are unique in the `states` list. This is currently not possible
with JMESPath.  In this example we can hard code the state `"WA"`:

```
states[?name==`"WA"`].cities
```

but it is not possible to base this on a value of `first_choice`, which
comes from the parent element.  This JEP proposes a solution that makes
this possible in JMESPath.

## Specification

The grammar will support a new token `$` that refers to the root of the
original input JSON document. The `$` token is inspired by the
[XPath](https://www.w3.org/TR/1999/REC-xpath-19991116) specification,
where the `/` token designates the root of the original XML document.

This JEP introduces the following productions:

```
root-node     = "$"
```

The `expression` production will also be updated like so:

```
expression    =/ root-node
```

### Motivating Example

With these changes defined, the expression in the “Motivation” section can be
be written as:

```
states[?name==$.first_choice].cities
```

Which evalutes to `["Seattle", "Bellevue", "Olympia"]`.

## Rationale

This JEP standardizes a common request when querying JSON document as seen in
[existing library](https://github.com/nanoporetech/jmespath-ts) implementations.
