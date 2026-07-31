# SAML hacking pipeline for Claude Code

This repo contains a minimal vulnerability pipeline aimed at finding bypasses in
SAML implementations. The basic entrypoint is:

```
/evaluate github.com/my-org/my-saml-library
```

This will create a directory under `./investigate` within the repo and being a
loop attempting to generate bypasses. It also ships a rudamentary `/report`
skill that can be used to produce a short markdown explination, and
self-contained proof-of-concept script.

The general outline of the pipeline is as follows:

- Clone and analyze the repo. Write a source annotation file that contains the
  entrypoints to SAML operations, and key checks and controls within the
  codebase.
- Loop:
  - Generate a corpus of "gadgets" - mechanisms for confusing or breaking the
    logic of an individual check.
  - Using those gadgets, hypothesize "findings" - end-to-end techniques that can
    bypass the SAML validation writ large.
  - Dedupe and throw out out-of-scope findings.
  - Prove or disprove the remaining findings by writing an full program that
    demonstrates the exploit.
  - Analyze findings and gadgets and determine if there are still interesting
    avenues to explore. If not, stop the loop.

The skill heavily uses sub-agents to reduce the token usage of the main
pipeline.

Source annotations, findings, and gadgets are all stored as `.jsonl` files to
allow Claude to quickly scan and modify enteries. Their schema is stored under
`./api`.

The `./knowledge` directory holds generalized information that can be shared
across agents including the definition of what's "in-scope", as well as primers
on XML digital signature validation.

## Vulnerabilities found

Authentication bypass:

- github.com/goauthentik/authentik
  - https://github.com/goauthentik/authentik/security/advisories/GHSA-35v6-hv2g-6992
- github.com/justinbleach/saml-client
  https://github.com/justinbleach/saml-client/issues/149
- github.com/litesaml/lightsaml
  - https://github.com/litesaml/lightsaml/security/advisories/GHSA-w553-pwx6-3mg9
- github.com/OneUptime/oneuptime
  - https://github.com/OneUptime/oneuptime/issues/2949

General signature bypasses (AuthnRequest, LogoutRequest, etc.):

- github.com/tngan/samlify
  - https://github.com/tngan/samlify/security/advisories/GHSA-63ch-w8pw-4mf2
- WSO2 Identity Server
- Ruby’s saml_idp
- Ory polis

DoS

- github.com/russellhaering/goxmldsig
  - Pre-auth quadratic memory allocation
  - Impacts all Go implementations
  - https://github.com/russellhaering/goxmldsig/security/advisories/GHSA-qhrp-hfff-vphr
- github.com/SAML-Toolkits/python3-saml
  - Pre-auth XSLT transform bomb
  - https://github.com/SAML-Toolkits/python3-saml/security/advisories/GHSA-8mmq-f9vf-8fg2
- github.com/IdentityPython/pysaml2
  - Pre-auth XSLT transform bomb
  - https://github.com/IdentityPython/pysaml2/security/advisories/GHSA-6qg6-h5xq-x84c
- github.com/xmldom/xmldom
  - Pre-auth quadratic memory allocation
  - Impacts samlify and node-saml
  - https://github.com/xmldom/xmldom/security/advisories/GHSA-965w-775f-mr7g
