# JSON Functions (GorillaStack)

|||
|---|---
| **JEP**    | 17
| **Author** | Chris Armstrong (Gorilla Stack)
| **Status** | draft
| **Created**| 31-10-2019

## Abstract

This document proposes adding functions for manipulating JSON data, including
a `parse_json` and `to_json` function.

## Motivation

There is separate requirements to both parse JSON strings and to serialise
full or partial object structures to JSON.

## Specification

The following functions will be added to the specification:

### parse_json

```
any parse_json(string $json_data)
```

Parse string `$json_data` containing JSON data and produce an object structure that can be
further manipulated by JMESPath.

If the string contains invalid JSON, an error should be generated.

### to_json

```
string to_json(any $value)
```

Serialise `$value` to a JSON string. No guarantees are made about formatting - this is
implementation dependent.

## Impact

Although this introduces no changes to existing syntax or functions, it may be challenging
to support correctly in implementations other than JavaScript if the underlying language
does not contain a spec-compliant JSON parser or producing arbitrary object structures for
use with JMESPath is difficult.
