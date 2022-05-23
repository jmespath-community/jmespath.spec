# Compact syntax for multi-select-hash

|||
|---|---
| **JEP**    | 14
| **Author** | Dan Vanderkam, Maxime Labelle
| **Status** | draft
| **Created**| 26-Feb-2022

## Abstract

This JEP proposes a grammar modification to support
[ES6](https://www.benmvp.com/blog/learning-es6-enhanced-object-literals/)-style
compact `multi-select-hash` constructs.

## Motivation

Consider the following JSON document:

```
{
  "values": [
    {"id": 1, "first": "Bill", "last": "Gates"},
    {"id": 2, "first": "Larry", "last": "Page"}
  ]
}
```

Let’s say we wanted to return a hash containing `id`s and `first` names.
This is currently possible using the following syntax:

```
values[*].{id:id, first:first}
```

Since `{x:x, y:y}` is such a common pattern, ES6 introduced compact object literal notation to simplify this. In ES6, `{x,y}` is equivalent to `{x:x, y:y}`.

This JEP supports this syntax to make JMESPath usage slightly less verbose.

## Specification

What is currently the `multi-select-hash` production will be renamed
to `multi-select-hash-keyval`:

```
multi-select-hash_keval = "{" ( keyval-expr *( "," keyval-expr ) ) "}"
```

The `multi-select-hash` production will be modified slightly to accomodate
the syntax changes:

```
multi-select-hash	= multi-select-hash-key
multi-select-hash	=/ multi-select-hash-keyval
```

The new `multi-select-hash-key` production is defined like so:

```
multi-select-hash-key = "{" ( identifier *( "," identifier ) ) "}" 
```

Here is the full update to the grammar:

```
multi-select-hash        = multi-select-hash-key
multi-select-hash        =/ multi-select-hash-keyval

multi-select-hash-key    = "{" ( identifier *( "," identifier ) ) "}" 
multi-select-hash-keyval = "{" ( keyval-expr *( "," keyval-expr ) ) "}"
```

### Motivating Example

With these changes defined, the expression in the “Motivation” section can be
be written as:

```
values[*].{id, first}
```

Which evaluates to `[{"id": 1, "first": "Bill"}, {"id": 2, "first": "Larry"}]`.

## Rationale

This JEP is inspired by [an idea](https://github.com/jmespath/jmespath.site/issues/43)
from Dan Vanderkam.
