# Analyzing source code

The goal of the analyze step is to determine what the entrypoints for SAML
response and other signature validation are. As well as key points in the code
that must be subverted in order for a bypass to be effective.

Take special care, and analyze documentation and code comments when claiming
that an entrypoint is actually an endorsed API. These APIs should pass SAML
bindings down to signature validation or encryption and be what a user would
actually use.

Use both `scope.md` and `xmldsig.md` to identify other interesting points in the
library. This should include places where canonicalization, signature
validation, element searches (e.g. xpath expressions), parsing, and attribute
extraction occur. Or how those options are configured.

If you find something interesting outside of those categories, please include
it.
