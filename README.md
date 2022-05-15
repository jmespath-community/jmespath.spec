# JMESPath Specification

Any changes to the JMESPath specification
(https://jmespath.site/specification.html) must have a JEP (JMESPath Enhancement
Proposal) which is tracked in this repo.  There are implementations of JMESPath
in over 9 different languages, and we want to make sure that any modifications
to the spec make sense for all JMESPath libraries.  A JEP helps to work through
the design process for new additions and ensures that the JMESPath community
has a chance to give feedback before it's officially part of the specification.

## Things that need a JEP

Any functionaly change that would require an update to the specification
(http://jmespath.site/specification.html) requires a JEP.

This includes, but is not limited to:

* New syntax
* New functions
* New semantics/functionality

## Things that do not need a JEP

Anything that is specific to a JMESPath library does not need a JEP.  You
should defer to the specific library's contributing guide.  This can include
additional language specific APIs, extension points (e.g. adding custom
functions), configuration options, etc.

## Guidelines for proposing new features

First, make sure that the feature has not been previously proposed.  If it has,
make sure to reference prior proposals and explain why this new proposal should
be considered despite similar proposals not being accepted.

Writing a JEP can be a lot of work, so it can help to get initial guidance
before getting too far.  You can chat on the JMESPath gitter channel
(https://gitter.im/jmespath/chat) to get an initial pulse of a new feature.

You can also create a discussion in this repo with a rough proposal
to get initial high level feedback. If you'd like to move
forward with the proposal you will still need to write a JEP
and send a pull request (PR).

### Tenets of JMESPath

When proposing new features, keep these tenets in mind.  Adhering to
these tenets gives your proposal a higher likelihood of being accepted:

* JMESPath is not specific to a particular programming language.  Avoid
  constructs that are difficult to implement in another language.
* JMESPath strives to have one way to do something.
* Features are driven from real world use cases.

# JMESPath Compliance Tests

This repo contains a suite of JMESPath compliance tests. JMESPath's implementations can use these tests in order to verify their
implementation adheres to the JMESPath spec.

## Test Organization

`grammar/*.yml` contains tests for general grammar functionality. These documents must validate against the `test_schema.yml`.

`functions/*.yml` contains a description of each function accompanied by a suite of tests/examples. These documents must validate against the `function_schema.yml`.

