# JMESPath Compliance Tests

This repo contains a suite of JMESPath compliance tests. JMESPath
implementations can use these tests in order to verify their
implementation adheres to the JMESPath spec.

# Test Organization

The `test/` directory contains JSON files containing the JMESPath
testcase. Each JSON file represents a JMESPath feature. Each JSON file
is a JSON list containing one or more tests suites:

    [
      <test suite 1>,
      <test suite 2>,
    ]

Each test suite is a JSON object that has the following keys:

  - `given` - The input data from which the JMESPath expression is
    evaluated.
  - `cases` - A list of test cases.
  - `comment` - An optional field containing a description of the test
    suite.

Each JMESPath test case can have the following keys:

  - `expression` - The JMESPath expression being tested.
  - `result` - The expected result from evaluating the JMESPath
    expression against the `given` input.
  - `error` - The type of error that should be raised as a result of
    evaluating the JMESPath expression. The valid values for an error
    are:
      - `syntax` - Syntax error from an invalid JMESPath expression.
      - `invalid-arity` - Wrong number of arguments passed to a
        function.
      - `invalid-type` - Invalid argument type for a function.
      - `invalid-value` - Semantically incorrect value (used in slice
        tests)
      - `unknown-function` - Attempting to invoke an unknown function.
  - `bench` - If the case is a benchmark, `bench` contains the type of
    benchmark. Available `bench` types are as follows:
      - `parse` - Benchmark only the parsing of an expression.
      - `interpret` - Benchmark only the interpreting of an expression.
      - `full` - Benchmark both parsing and interepreting an expression.
  - `comment` - An optional comment containing a description of the
    specific test case.

For each test case, either `result`, `error`, or `bench` must be
specified. Only one of these keys can be present in a single test case.

The error type (if the `error` key is present) indicates the type of
error that an implementation should raise, but it does not indicate
**when** this error should be raised. For example, a value of `"error":
"syntax"` does not require that the syntax error be raised when the
expression is compiled. If an implementation does not have a separate
compilation step this won't even be possible. Similar for type errors,
implementations are free to check for type errors during compilation or
at run time (when the parsed expression is evaluated). As long as an
implementation can detect that this error occurred at any point during
the evaluation of a JMESPath expression, this is considered sufficient.

Below are a few examples:

    [{
        "given":
            {"foo": {"bar": {"baz": "correct"}}},
         "cases": [
             {
                "expression": "foo",
                "result": {"bar": {"baz": "correct"}}
             },
             {
               "expression": "foo.1",
               "error": "syntax"
             },
        ]
    }]

This above JSON document specifies 1 test suite that contains 2 test
cases. The two test cases are:

  - Given the input `{"foo": {"bar": {"baz": "correct"}}}`, the
    expression `foo` should have a result of `{"bar": {"baz":
    "correct"}}`.
  - Given the input `{"foo": {"bar": {"baz": "correct"}}}`, the
    expression `foo.1` should generate a syntax error.

# Utility Tools

Most languages have test frameworks that are capable of reading the JSON
test descriptions and generating testcases. However, a `jp-compliance`
tool is provided to help with any implementation that does not have an
available test framework to generate test cases. The `jp-compliance`
tool takes the name of a jmespath executable and will evaluate all the
compliance tests using this provided executable. This way all that's
needed to verify your JMESPath implementation is for you to write a
basic executable. This executable must have the following interface:

  - Accept the input JSON data on stdin.
  - Accept the jmespath expression as an argument.
  - Print the jmespath result as JSON on stdout.
  - If an error occurred, it must write the error name to sys.stderr.
    This check is case-insensitive. The error types in the compliance
    tests are hyphenated, but each individual component may appear in
    stderr (again case-insensitive).

Here are a few examples of error messages that would pass
`jp-compliance`:

  - Error type: `unknown-function`
  - Valid error messages:
      - `unknown-function: somefunction()`
      - `error: unknown function 'somefunction()`
      - `Unknown function: somefunction()`
  - Error type: `syntax`
  - Valid error messages:
      - `syntax: Unknown token '$'`
      - `syntax-error: Unknown token '$'`
      - `Syntax error: Unknown token '$'`
      - `An error occurred: Syntax error, unknown token '$'`

## Note

This will be substantially slower than using a test framework. Using
`jp-compliance` each test case is evaluated by executing a new process.

You can run the `/bin/jp-compliance --help` for more information and for
examples on how to use this tool.