# Lexical Scoping

|||
|---|---
| **JEP**      | 11a 
| **Author**   | James Saryerwinnie
| **Status**   | accepted
| **Created**  | 21-March-2023
| **Obsoletes**| [JEP-011](./jep-011-let-function.md)

> This proposal tracks [JEP-018](https://github.com/jmespath/jmespath.jep/blob/main/proposals/0018-lexical-scope.md) proposal from jmespath.org.

## Abstract

[abstract]: #abstract

This JEP proposes the introduction of lexical scoping using a new
`let` expression.  You can now bind variables that are evaluated in the
context of a given lexical scope.  This enables queries that can refer to
elements defined outside of their current element, which is not currently
possible.  This JEP supercedes JEP 11, which proposed similar functionality
through a `let()` function.

## Motivation
[motivation]: #motivation

A JMESPath expression is always evaluated in the context of a current
element, which can be explicitly referred to via the `@` token.  The
current element changes as expressions are evaluated.  For example,
suppose we had the expression `foo.bar[0]` that we want to evalute against
an input document of:

```json
{"foo": {"bar": ["hello", "world"]}, "baz": "baz"}
```

The expression, and the associated current element are evaluated as follows:

```
# Start
expression = foo.bar[0]
@ = {"foo": {"bar": ["hello", "world"]}, "baz": "baz"}

# Step 1
expression = foo
@ = {"foo": {"bar": ["hello", "world"]}, "baz": "baz"}
result = {"bar": ["hello", "world"]}

# Step 2
expression = bar
@ = {"bar": ["hello", "world"]}
result = ["hello", "world"]

# Step 3
expression = [0]
@ = ["hello", "world"]
result = "hello"
```

The end result of evaluating this expression is `"hello"`.  Note that each
step changes the values that are accessible to the current expression being
evaluated.  In "Step 2", it is not possible for the expression to reference
the value of `"baz"` in the current element of the previous step, "Step 1".

This ability to reference variables in a parent scope is a serious limitation
of JMESPath, and anecdotally is one of the commonly requested features
of the language.  Below are examples of input documents and the desired output
documents that aren't possible to create with the current version of
JMESPath:

```
Input:

[
  {"home_state": "WA",
   "states": [
     {"name": "WA", "cities": ["Seattle", "Bellevue", "Olympia"]},
     {"name": "CA", "cities": ["Los Angeles", "San Francisco"]},
     {"name": "NY", "cities": ["New York City", "Albany"]}
   ]
  },
  {"home_state": "NY",
   "states": [
     {"name": "WA", "cities": ["Seattle", "Bellevue", "Olympia"]},
     {"name": "CA", "cities": ["Los Angeles", "San Francisco"]},
     {"name": "NY", "cities": ["New York City", "Albany"]}
   ]
  }
]


(for each list in "states", select the list of cities associated
 with the state defined in the "home_state" key)

Output:

[
  ["Seattle", "Bellevue", "Olympia"],
  ["New York City", "Albany"]
]
```

```
Input:
{"imageDetails": [
  {
    "repositoryName": "org/first-repo",
    "imageTags": ["latest", "v1.0", "v1.2"],
    "imageDigest": "sha256:abcd"
  },
  {
    "repositoryName": "org/second-repo",
    "imageTags": ["v2.0", "v2.2"],
    "imageDigest": "sha256:efgh"
  },
]}


(create a list of pairs containing an image tag and its associated repo name)

Output:

[
  ["latest", "org/first-repo"],
  ["v1.0", "org/first-repo"],
  ["v1.2", "org/first-repo"],
  ["v2.0", "org/second-repo"],
  ["v2.2", "org/second-repo"],
]
```

In order to support these queries we need some way for an expression to
reference values that exist outside of its implicit current element.


## Specification
[specification]: #specification

A new "let expression" is added to the language.  The expression has the
format: `let <bindings> in <expr>`.  The updated grammar rules in ABNF are:

```
let-expression = "let" bindings "in" expression
bindings = variable-binding *( "," variable-binding )
variable-binding = variable-ref "=" expression
variable-ref = "$" unquoted-string
```

The `let-expression` and `variable-ref` rule are also added as a new expression
types:

```
expression =/ let-expression / variable-ref
```

Examples of this new syntax:

* `let $foo = bar in {a: myvar, b: $foo}`
* `let $foo = baz[0] in bar[? baz == $foo ] | [0]`
* `let $a = b, $c = d in bar[*].[$a, $c, foo, bar]`

It's worth noting that this is the first JEP to introduce keywords into the
language: the `let` and `in` keywords.  These are not reserved keywords, these
words can continue to be used as identifiers in expressions.  There are no
backwards incompatible changes being proposed with this JEP. The grammar rules
unambiguously describe whether ``let`` is meant to be interpreted as a keyword
or as an identifier (often referred to as contextual keywords).

### New evaluation rules

Let expressions are evaluated as follows.

Given the rule `"let" bindings "in" expression`, the `bindings` rule is
processed first.  Each `variable-binding` within the `bindings` rule defines
the name of a variable and an expression. Each expression is evaluated, and the
result of this evaluation is then bound to the associated variable name.

Once all the `variable-binding` rules have been processed, the associated
`expression` clause of the let expression is then evaluated.  During the
evaluation of the expression, any references, via the `variable-ref` rule, to a
variable name will evaluate to the value bound to the name.  Once the
associated expression has been evaluated, the let expression itself evaluates
to the result of this expression.  After the let expression has been evaluated,
the variable bindings associated with the let expression are no longer valid.
This is also referred to as the visibility of a binding; the bindings of a
let expression are only visible during the evaluation of the `expression`
clause of the let expression.

When evaluating the `bindings` rule, a `variable-binding` for a variable name
that is already visible in the current scope will replace the existing binding
when evaluating the `expression` clause of the let expression.  This means in
the context of nested let expressions (and consequently nested scopes), a
variable in an inner scope can shadow a variable defined in an outer scope.

If a `variable-ref` references a variable that has not been defined, the
evaluation of that `variable-ref` will trigger an `undefined-variable` error.
This error MUST occur when the expression is evaluated and not at compile
time.  This is to enable implementations to define an implementation specific
mechanism for defining an initial or "global" scope.  Implementations are free
to offer a "strict" compilation mode that a user can opt into, but MUST support
triggering an `undefined-variable` error only when the `variable-ref` is
evaluated.

Note that when evaluating the `bindings` rule, the expression bound
to a variable is completely evaluated before binding to the variable.
Any references to the variable are replaced with the result of this evaluation,
the expression is not re-evaluated.  This is worth clarifying specifically
for projections (wildcard expressions, the flatten operator, slices and
filter expressions).  If the expression being bound is a projection, the
evaluation of this expression effectively stops the projection.  This means
subsequent references using the `variable-ref` MUST NOT continue projecting
to child expressions.  For example, this is the behavior for a projection:

```
search(
  foo[*][0]
  {"foo": [[0, 1], [2, 3], [4, 5]]}
) -> [0, 2, 4]
```

And this is the behavior when assigning a variable to a projection:

```
search(
  let $foo = foo[*]
  in
    $foo[0]
  {"foo": [[0, 1], [2, 3], [4, 5]]}
) -> [0, 1]
```

In the first example, the `[0]` expression is projected onto each element
in the list, returning the first element of each sub list: `[0, 2, 4]`.
In the second example, the `foo[*]` expression is evaluated to
`[[0, 1], [2, 3], [4, 5]]` and assigned to the variable `$foo`.  The
projection expression evaluation is complete, and the projection is stopped.
Evaluating the expression `$foo[0]` results in the variable `$foo` being
replaced with its bound value of `[[0, 1], [2, 3], [4, 5]]`, so the entire
expression becomes `[[0, 1], [2, 3], [4, 5]][0]`, which returns the first
element in the list which is `[0, 1]`.

### Examples

Basic examples demonstrating core functionality.

```
search(let $foo = foo in $foo, {"foo": "bar"}) -> "bar"
search(let $foo = foo.bar in $foo, {"foo": {"bar": "baz"}}) -> "baz"
search(let $foo = foo in [$foo, $foo], {"foo": "bar"}) -> ["bar", "bar"]
```

Nested bindings.

```
search(
  let $a = a
  in
    b[*].[a, $a, let $a = 'shadow' in $a],
  {"a": "topval", "b": [{"a": "inner1"}, {"a": "inner2"}]}
) -> [["inner1", "topval", "shadow"], ["inner2", "topval", "shadow"]]
```

Error cases.

```
search($foo, {}) -> <error: undefined-variable>
search([let $foo = 'bar' in $foo, $foo], {}) -> <error: undefined-variable>
```


## Rationale
[rationale]: #rationale

*Note: see [previous discussion](https://github.com/jmespath/jmespath.site/pull/6#issuecomment-1464113092)
for more background.*

### Introducing keywords into the language

The let expression proposed in this JEP is based off of similar constructs
in existing programming languages:

* [Haskell](http://learnyouahaskell.com/syntax-in-functions#let-it-be)
* [Clojure](https://clojuredocs.org/clojure.core/let)
* [OCaml](https://v2.ocaml.org/manual/expr.html#sss:expr-localdef)

It was important to borrow from existing syntax and semantics.
Lexical scoping is a familiar concept to developers, so care was
taken to be consistent with the mental model that developers already have.

Alternatives were considered that avoided introducing new keywords into
the language (this proposal adds the first keyword to the language).  These
included some variation that approximated defining an anonymous function
with arguments, e.g.:

```
|foo, bar| => {$foo: a, $bar: b}
```

The reason for not going with this approach is that adding the ability
to define functions is a large feature that will take considerable effort
to design.  This may be something to consider in the future, but it's
a larger scope than introducing lexical scoping and made the most sense
to address separately.  We'd also need to introduce not only defining
anonymous functions with arguments, but also a mechanism to invoke
such functions.  You can then create lexical scope by defining a function
and immediately invoking it.  For example, in javascript it would look
like this:

```typescript
(({x, y}) => ([x, y]))(
    {x: "foo", y: "bar"}
);
```

This was considered too verbose for such a common use case of defining
variables.  It makes sense that a dedicated, more succinct syntax was
preferred, as many languages have a dedicated `let` syntax for defining
variables.

#### Backwards compatibility concern

Languages will often design keywords as reserved words that can't be used
as variable names or other identifiers.  This helps to provide clarity
because the reader knows that the keyword can only have a single meaning.
This is possible to do when you first design the language, or if you are
willing to introduce breaking changes into the language.  JMESPath instead
takes an alternate approach of introducing keywords that can be inferred
from the context in which they're used, which is known as contextual
keywords.  There are other languages that also take this approach,
`such as C# <https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/#contextual-keywords>`__.

In order to do this, the updated grammar rules must be chosen to avoid any
ambiguity when parsing expressions.  This may limit the syntax and location
where the new keywords could be used, so the tradeoffs of adding a new keyword
must be considered carefully.

We should be wary of adding new keywords to JMESPath, and only do so when
there is a strong rationale for doing so. ``let`` is one such case, as detailed
in this section.

### Adding a sigil for variable references

One of the changes from an earlier proposal of this feature (JEP-11) is that
this proposal adds explicit syntax for variable references via the `$foo`
syntax.  The lookup process between the expression `foo` and `$foo` are
fundamentally different types of lookup.  One searches through values
from the implicit current element and one is a lookup in the lexical scope.

Not having a syntactic difference creates ambiguity regarding the intended
type of lookup by the user.  This also prevents defining where scoped lookups
are allowed through the grammar.  For example, in the expression `foo.bar`
it is unclear whether `bar` refers to a lookup in the current element or the
lexical scope.  Having explicit syntax removes this ambiguity, allowing a user
to explicitly state their intent.  It also enables distinct error conditions.

A reference to a non-existent variable is an error, as the user provided
explicit syntax stating that they expect the variable to exist.  The variable
not existing is result of the user not binding the variable name at some
point, which is an error.  Conversely, an expression evaluated against the
current element results in `null` if the key does not exist.  This is
because the query is being evaluated against an input JSON document, and
we don't know what keys may or may not be present.

## Multiple assignments with commas

Assigning multiple variables is done through comma separated `variable-binding`
rules, e.g. `$foo = foo, $bar = bar`.  An alternative considered was to use
syntax similar to Javascript's object destructuring:

```
let {$foo, $bar} = @ in ...
```

There are several reasons this alternative was not chosen:

* This requires multiple assignments to come from an object type, which
  might require a user to unnecessarily create a multi-select-hash in order
  to assign multiple variables.
* Destructuring binds to top level values in an object, and does not allow
  for a single binding to evaluate to an expression, without having to again
  preconstruct that value via a multi-select-hash.
* Object destructuring is an additive change.  Nothing in this JEP precludes
  this addition in the future, e.g.:

```
let {$foo, $bar} = @ in ...
let {$foo, $bar} = @, $baz = a.b.c in ...
```

## Unbound values error at evaluation time not at compile time

The JEP also requires that unbound values error at evaluation time, not
at compile time.  This enables implementations to bind an initial (global)
scope when a query is evaluated.  This is something that other query languages
provide, and is useful to define fixed queries that only vary by the value
of specific variables.  We'll look at a few examples.

First, consider a command line utility, let's called it `jp`, that accepts a
path to a file containing a JMESPath query and reads an input JSON document
through stdin.  This command line utility could offer a `--params` option that
allows a user to pass in an initial scope.  For example:

*myquery.jmespath*

```
results[*].[name, uuid, $hostname]
```

A user could then use this CLI to retrieve JSON data and filter it:

```
$ curl https://myapi/info/$HOSTNAME | \
    jp --filename myquery.jmespath --params '{"hostname": "$HOSTNAME"}'
```

In this case the JMESPath expression does not need to change and can be
shared with other people, and still include data that's specific to your
machine.

Another example would be where JMESPath is used in some shared definition file.
Suppose we had a file that defines how to make an API request, and specifies a
condition we'd like to meet based on the output response.  We want to describe
that the expected output depends on the input provided.  This is how we can
describe this:

```json
{"GroupActive": {
   "operation": "DescribeGroups",
   "acceptors": {
     "argument": "Response[].[length(Instances[?State=='Active']) == length($params.GroupNames)",
     "matcher": "path"
   }
}}
```

This is saying that we should invoke the `DescribeGroups` operations with a
list of group names, and that we want to check that the response contains a
list of `Instances` with `State == 'Active'` whose length matches the length of
the params group names.  You could now bind the user provided params as the
initial scope of `{"params": inputParams}` and code generate something like
this (using the python JMESPath library in this example):

```python
def wait(user_params):
  response = client.DescribeGroups(user_params)
  expected = jmespath.compile(
    "Response[].[length(Instances[?State=='Active']) "
    "== length($params.GroupNames)"
  )
  result = expected.search(
    response,
    # This is the new part, give queries access to the user params
    # via the $params variable.
    scope={'params': user_params},
)
  if result:
    return "SomeSuccessResponse"
  return "SomeFailureResponse"

# User can invoke this via:
wait({"GroupNames": ["group1", "group2", "group3"]})
```

This JEP does not require that implementations provide this capability of
passing in an initial scope, but by requiring that undefined variable
references are runtime errors it enables implementations to provide this
capability.  Implementations are also free to provide an opt-in "strict"
mode that can fail at compile time if a user knows they will not be providing
an initial scope.

## Testcases
[testcases]: #testcases

Basic expressions

```yaml
# Basic expressions
- given:
    foo:
      bar: baz
  cases:
    - expression: "let $foo = foo in $foo"
      result:
        bar: baz
    - expression: "let $foo = foo.bar in $foo"
      result: "baz"
    - expression: "let $foo = foo.bar in [$foo, $foo]"
      result: ["baz", "baz"]
    - comment: "Multiple assignments"
      expression: "let $foo = 'foo', $bar = 'bar' in [$foo, $bar]"
      result: ["foo", "bar"]
# Nested expressions
- given:
    a: topval
    b:
      - a: inner1
      - a: inner2
  cases:
    - expression: "let $a = a in b[*].[a, $a, let $a = 'shadow' in $a]"
      result:
        - ["inner1", "topval", "shadow"]
        - ["inner2", "topval", "shadow"]
    - comment: Bindings only visible within expression clause
      expression: "let $a = 'top-a' in let $a = 'in-a', $b = $a in $b"
      result: "top-a"
# Let as valid identifiers
- given:
    let:
      let: let-val
      in: in-val
  cases:
    - expression: "let $let = let in {let: let, in: $let}"
      result:
        let:
          let: let-val
          in: in-val
        in:
          let: let-val
          in: in-val
    - expression: "let $let = 'let' in { let: let, in: $let }"
      result:
        let:
          let: let-val
          in: in-val
        in: "let"
    - expression: "let $let = 'let' in { let: 'let', in: $let }"
      result:
        let: "let"
        in: "let"
# Projections stop
- given:
    foo: [[0, 1], [2, 3], [4, 5]]
  cases:
    - comment: Projection is stopped when bound to variable
      expression: "let $foo = foo[*] in $foo[0]"
      result: [0, 1]
# Examples from Motivation section
- given:
    - home_state: WA
      states:
        - name: WA
          cities: ["Seattle", "Bellevue", "Olympia"]
        - name: CA
          cities: ["Los Angeles", "San Francisco"]
        - name: NY
          cities: ["New York City", "Albany"]
    - home_state: NY
      states:
        - name: WA
          cities: ["Seattle", "Bellevue", "Olympia"]
        - name: CA
          cities: ["Los Angeles", "San Francisco"]
        - name: NY
          cities: ["New York City", "Albany"]
  cases:
    - expression: "[*].[let $home_state = home_state in states[? name == $home_state].cities[]][]"
      result:
        - ["Seattle", "Bellevue", "Olympia"]
        - ["New York City", "Albany"]
- given:
    imageDetails:
      - repositoryName: "org/first-repo"
        imageTags:
          - latest
          - v1.0
          - v1.2
        imageDigest: "sha256:abcd"
      - repositoryName: "org/second-repo"
        imageTags:
          - v2.0
          - v2.2
        imageDigest: "sha256:efgh"
  cases:
    - expression: >
        imageDetails[].[
          let $repo = repositoryName,
              $digest = imageDigest
          in
            imageTags[].[@, $digest, $repo]
        ][][]
      result:
        - ["latest", "sha256:abcd", "org/first-repo"]
        - ["v1.0", "sha256:abcd", "org/first-repo"]
        - ["v1.2", "sha256:abcd", "org/first-repo"]
        - ["v2.0", "sha256:efgh", "org/second-repo"]
        - ["v2.2", "sha256:efgh", "org/second-repo"]
# Errors
- given: {}
  cases:
    - expression: "$noexist"
      error: "undefined-variable"
    - comment: Reference out of scope variable
      expression: "[let $scope = 'foo' in [$scope], $scope]"
      error: "undefined-variable"
    - comment: Can't use var ref in RHS of subexpression
      expression: "foo.$bar"
      error: "syntax"
```
