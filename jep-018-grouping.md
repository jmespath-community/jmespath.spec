# group_by function

|||
|---|---
| **JEP**    |  18
| **Author** | Maxime Labelle
| **Created**| 05-August-2022
| **SemVer** | MINOR
| **Status**| Draft

## Abstract

This JEP introduces a new `group_by()` function.

## Specification

### group_by

```
object group_by(array[object] $elements, expression->string) $expr)
```

Groups an array of objects `$elements` using an expression `$expr` as the group key.
The `$expr` expression is applied to each element in the array `$elements` and the 
resulting value is used as a group key.

The result is an object whose keys are the unique set of string keys and whose respective values are an array of objects array matching the group criteria.

Objects that do not match the group criteria are discarded from the output.
This includes objects for which applying the `$expr` expression evaluates to `null`.

If the result of applying the `$expr` expression against the current array element
results in type other than `string` or `null`, a type error MUST be raised.

### Examples

Given the following input JSON document:

```json
{
  "items": [
    { "spec": { "nodeName": "node_01", "other": "values_01" } },
    { "spec": { "nodeName": "node_02", "other": "values_02" } },
    { "spec": { "nodeName": "node_03", "other": "values_03" } },
    { "spec": { "nodeName": "node_01", "other": "values_04" } }
  ]
}
```

Example:

|Expression|Result
|---|---
|`` group_by(items, &spec.nodeName) ``| ` {"node_01": [{"spec": {"nodeName": "node_01", "other": "values_01"}}, {"spec": {"nodeName": "node_01", "other": "values_04"}}], "node_02": [{"spec": {"nodeName": "node_02", "other": "values_02"}}], "node_03": [{"spec": {"nodeName": "node_03", "other": "values_03"}}]} `

Given the following input JSON document:

```json
{
  "array": [
    { "name": "one", "b": true },
    { "name": "two", "b": false },
    { "b": false }
  ]
}
```

Here are a couple of other examples:

|Expression|Result
|---|---
|`` group_by(array, &name) `` | ` {"one":[{"name":"one","b":true}],"two":[{"name":"two","b":false}]} `
|`` group_by(array, &b) `` | `invalid-type`
