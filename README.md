# SAML hacking pipeline for Claude Code

This repo contains a minimal vulnerability pipeline aimed at finding bypasses in
SAML implementations. You can read more about this at:
https://oblique.security/blog/hacking-saml.

The basic entrypoint is:

```
/evaluate github.com/my-org/my-saml-library
```

This will create a directory under `./investigate` within the repo and being a
loop attempting to generate bypasses. It also ships a rudamentary `/report`
skill that can be used to produce a short markdown explination, and
self-contained proof-of-concept script.

Source annotations, findings, and gadgets are all stored as `.jsonl` files to
allow Claude to quickly scan and modify enteries. Their schema is stored under
`./api`.

The `./knowledge` directory holds generalized information that can be shared
across agents including the definition of what's "in-scope", as well as primers
on XML digital signature validation.

## Vulnerabilities found

Currently published issues (more will be added as reports become public).

Authentication bypass:

- github.com/goauthentik/authentik ([CVE-2026-57580](https://github.com/goauthentik/authentik/security/advisories/GHSA-35v6-hv2g-6992))
- github.com/litesaml/lightsaml ([CVE-2026-63182](https://github.com/litesaml/lightsaml/security/advisories/GHSA-w553-pwx6-3mg9))
- github.com/OneUptime/oneuptime ([#2949](https://github.com/OneUptime/oneuptime/issues/2949), [#2981](https://github.com/OneUptime/oneuptime/issues/2981), [#2988](https://github.com/OneUptime/oneuptime/issues/2988))
- github.com/justinbleach/saml-client ([#149](https://github.com/justinbleach/saml-client/issues/149))

DoS:

- github.com/russellhaering/goxmldsig ([GHSA-qhrp-hfff-vphr](https://github.com/russellhaering/goxmldsig/security/advisories/GHSA-qhrp-hfff-vphr))
- github.com/SAML-Toolkits/python3-saml ([#447](https://github.com/SAML-Toolkits/python3-saml/issues/447))
- github.com/IdentityPython/pysaml2 ([#1035](https://github.com/IdentityPython/pysaml2/issues/1035))

There are a number of outstanding reports for signature bypasses for AuthnRequests, AttributeQuery, and LogoutRequest objects. These will be added here as they become public.
