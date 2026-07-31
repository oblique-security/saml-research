# Vulnerability scopes

## In scope

This pipeline is solely aimed at bypassing signature validation on SAML
entities or confusing a SAML binding to achieve a similar outcome.

All findings should result in an attacker with a signed document from the IdP
(or no document at all), being able to affect the results of the way the library
reads values from that XML document.

The primary focus should be on bypassing signature validation of SAML responses
to change attribute or name ID values.

As a secondary focus, attempt to bypass LogoutResponse or AuthnRequest
signatures as well (or any other message that is signed within the SAML
protocol).

## Out of scope

- Dangerous config options that are not enabled by default are out of scope.
  Reports should never be conditioned on behavior like "if a user fails to
  provide certificates, then the signature isn't checked."
- Lack of signature checks. If a library doesn't implement LogoutRequest
  signature validation, don't report this as a finding.
- IdPs providing weird values within signed elements. Never investigate "what if
  a valid user were to set a weird unicode value in their email." Always assume
  that the IdP sanitizes values. Though an attacker modifying a signed attribute
  after the fact is fair game.
- DoS. Don't investigate denial of service vectors.
- Replay prevention or inResponseTo validation issues. Being able to submit a
  valid response twice isn't interesting.
- Modifying values that aren't required to be signed. If the library accepts
  signed Assertions sub-elements rather than top level Responses, modifying
  the Response Destination isn't interesting.
