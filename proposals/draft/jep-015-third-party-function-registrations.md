# Third-Party Functions

|||
|---|---
| **JEP**    | 15
| **Author** | Nolan Wood, Maxime Labelle
| **Status** | draft
| **Created**| 28-Feb-2022

## Abstract

This JEP proposes a new distinction between _reserved function names_
and _third-party function names. This allows the spec to evolve by
introducing new functions without the fear of clashing with existing
functions from popular third-party implementations.

## Motivation

Virtually all the current language implementations of JMESPath support
registering additional "custom" functions that add features to JMESPath.

There is currently no restrictions as to what those function names may be
except that is must match the `unquoted-string` production.

This proposal specifically acknowledges that third-party functions may
be used to extend JMESPath and mandates that all third-party function
names should start with a leading underscore character.

Functions whose names – known or future – starts with a letter are
effectively reserved exclusively for use by JMESPath.

## Specification

This JEP proposes adding the following text to the specification in the
[Function Expressions](https://jmespath-unofficial.github.io/jmespath.site/specification.html#functions) section:

> ### Function Expressions
> Functions allow users to easily transform and filter data in JMESPath expressions.
> Built-in functions are defined by JMESPath. However, a compliant implementation
> MAY support registration of new functions to extend JMESPath.
>
> Functions whose name starts with an alphabetic character (_i.e_ `A-Za-z`) are reserved
> for JMESPath.
> Conversely, third-party function names SHOULD start with a leading underscore character
> (_i.e_ `_` ).
> 
> Registration of conflicting function names MAY raise an error.
> However, if no error is raised, built-in functions MUST take precedence over
> third-party functions.
> The resolution of conflicting function names from competing third-party libraries
> is undefined.


## Rationale

This JEP does not introduce any grammar change.

Although there has been discussions about having a different semantic constructs
for both _reserved functions_ and _third-party functions_, it has been considered
as an implementation detail. Besides, this would unnecessarily make it more
difficult for library authors to maintain two different logics for evaluating
functions.
