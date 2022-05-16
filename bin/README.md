# JMESPath Compliance Tests

This repo contains a suite of JMESPath compliance tests. JMESPath's implementations can use these tests in order to verify their
implementation adheres to the JMESPath spec.

## Test Organization

`grammar/*.yml` contains tests for general grammar functionality. These documents must validate against the `grammar_schema.yml`.

`functions/*.yml` contains a description of each function accompanied by a suite of tests/examples. These documents must validate against the `function_schema.yml`.

## Running Tests

This repository include a Python `jp-compliance` executable test runner.

```
jp-compliance <executable> <test yaml> [<test name>..]
```

This will run the JMESPath compliance tests against a JMESPath executable.
The executable must accept the query as the only argument, and the input data on stdin.
The result must be printed to stdout as JSON.
Errors must be printed to stderr and match expected values.

If your executable requires additional arguments, wrap it in an executable script.

The test YAML must validate against function_schema.yml or test_schema.yml.

Additionally, test names can be supplied to only execute matching tests