This project exists as the original implementation does not appear to be maintained and has not received any improvements for a long time. After multiple [attempts](https://github.com/jmespath/jmespath.site/issues/65) to [contact](https://github.com/jmespath/jmespath.site/issues/94) the original author, we decided to proceed further with this project.

# JMESPath Specification

Any changes to the JMESPath specification
(https://jmespath.site/specification.html) must have a JEP (JMESPath Enhancement Proposal) which is tracked in this repo. There are implementations of
JMESPath in over 9 different languages, and we want to make sure that any modifications to the spec make sense for all JMESPath libraries. A JEP helps
to work through the design process for new additions and ensures that the JMESPath community has a chance to give feedback before it's officially part
of the specification.

## Things that need a JEP

Any functionality change that would require an update to the specification
(http://jmespath.site/specification.html) requires a JEP.

This includes, but is not limited to:

* New syntax
* New functions
* New semantics/functionality

## Things that do not need a JEP

Anything that is specific to a JMESPath library does not need a JEP. You should defer to the specific library's contributing guide. This can include
additional language specific APIs, extension points (e.g. adding custom functions), configuration options, etc.

## Guidelines for proposing new features

First, make sure that the feature has not been previously proposed. If it has, make sure to reference prior proposals and explain why this new
proposal should be considered despite similar proposals not being accepted.

Writing a JEP can be a lot of work, so it can help to get initial guidance before getting too far. You can chat on the JMESPath gitter channel
(https://gitter.im/jmespath/chat) to get an initial pulse of a new feature.

Then open a [discussion](https://github.com/jmespath-community/jmespath.spec/discussions) to discuss the feature, its merit, and any possible
alternatives. Once the discussion has reached a mature level of feedback it will be assigned a JEP#. At this point it is safe to begin composing a
JEP, along with all the required changes and documentation.

### Tenets of JMESPath

When proposing new features, keep these tenets in mind. Adhering to these tenets gives your proposal a higher likelihood of being accepted:

* JMESPath is not specific to a particular programming language. Avoid constructs that are difficult to implement in another language.
* JMESPath strives to have one way to do something.
* Features are driven from real world use cases.
* Syntax is generally used to express data structure manipulation while functions are used to express datum manipulation

# JMESPath Compliance Tests

This repo contains a suite of JMESPath compliance tests. JMESPath's implementations can use these tests in order to verify their implementation
adheres to the JMESPath spec. Every implementation of JMESPath should provide a basic command line executable that allows execution of compliance
tests. This eases third-party validation of the implementation, without needing any specific testing suite.

See [bin/README.md](bin/README.md) for more information on running compliance tests.
