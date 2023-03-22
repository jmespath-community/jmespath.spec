# Add 'error' function

|||
|---|---
| **JEP**    |  20
| **Author** | Sebastien Rosset
| **Created**| 22-March-2023
| **[SemVer](https://semver.org/spec/v2.0.0.html#summary)** | MINOR

## Abstract

This JEP proposes a new function `error()` that raises a runtime error.
This could be used to detect and raise an error when input JSON documents have unexpected data.


## Motivation

AFAIK, there is no dedicated function to raise a runtime error within a JMESpath expression.


## Specification

Add a new `error` function that takes a string expression.
If the `error` function is evaluated, an error is raised with the specified string message.

## Rationale

Suppose we have the following JSON documents:
```json
{ "id": "eth0", "status": "up" }
{ "id": "eth1", "status": "down" }
```

In this example, the values of `status` are either `up` or `down`.
The values need to be mapped to `1` and `0`, which can be done using a JMESpath expression:

```
((status == 'up') && `1`) || ((status == 'down') && `0`) || 0
```

The user might want to raise an error when unexpected input is encountered, such as when the value of `status` is set to `degraded`:
```json
{ "id": "eth2", "status": "degraded" }
```

Since there is no built-in function to raise error, one may rely on the fact that some JMESpath expressions are invalid, e.g., `to_number('bad')` would raise an error, but this is kludgy.
```
((status == 'up') && `1`) || ((status == 'down') && `0`) || to_number('bad')
```

## Implementation Notes

Are there any details that implementers should try to follow?
