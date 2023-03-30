This project is a community initiative that aims to discuss frequently requested features for JMESPath.
It tracks updates to the original specification and brings improvements that may not be necessarily included in the original spec.
This project supports an ecosystem of high quality, fast paced implementations in popular programming languages.

# JMESPath Specification

Any changes to the [JMESPath specification](https://jmespath.site/main/#specification) must have a JEP (JMESPath Enhancement Proposal) which is tracked in this repo. While there are implementations of JMESPath in over 9 different languages, we currently track implementations of 4 implementations of the JMESPath Community specification. We want to make sure that any modifications to the spec make sense for all JMESPath and JMESPath Community libraries.

A JEP helps to work through the design process for new additions and ensures that the JMESPath community has a chance to give feedback before it's officially part of the specification.

## Things that need a JEP

Any functionality change that would require an update to the specification
(https://jmespath.site/main/#specification) requires a JEP.

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

Then open a [discussion](https://github.com/jmespath-community/jmespath.spec/discussions) to discuss the feature, its merit, and any possible alternatives.
Once the discussion has reached a mature level of feedback, it will be assigned a JEP number _N_.
At that point it is safe to begin composing JEP-_N_ along with all the required changes and documentation.

In rare circumstances, JEPs will be superceded by newer versions of a JEP. Those
updated JEPs will have an amended letter sequence added as a suffix to the original JEP#.
The sequence follows the 26 US-ASCII alphabetical order 'a', 'b', to 'z'. It then follows with
'aa', 'ab', to 'az' and so on.

For instance, [JEP-12](./jep-012-raw-string-literals.md) introduced `raw-string`
literals but the grammar defined there did not match that which was eventually specified. An update has been published as
[JEP-12a](./jep-012a-raw-string-literals.md).

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
