# XMLDSig

This file describes SAML signature validation. The following is an example signed
SAML document:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<samlp:Response xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" ID="_response_id" Version="2.0" IssueInstant="2024-01-01T00:00:00Z" Destination="https://sp.example.com/acs">
  <saml:Issuer>https://idp.example.com</saml:Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
  </samlp:Status>
  <saml:Assertion ID="_assertion_id" Version="2.0" IssueInstant="2024-01-01T00:00:00Z">
    <saml:Issuer>https://idp.example.com</saml:Issuer>
    <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
      <ds:SignedInfo>
        <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
        <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
        <ds:Reference URI="#_assertion_id">
          <ds:Transforms>
            <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
            <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
          </ds:Transforms>
          <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
          <ds:DigestValue>CHSa2Xa8kwNX+spYBsvHz+2ObvGGg+QqfoSxZauNiEU=</ds:DigestValue>
        </ds:Reference>
      </ds:SignedInfo>
      <ds:SignatureValue>l9xNY9pljp6tTORI6AIiornW/NkNUxkEvKvyK8iZG8vAk8slG7wlJKc4cd4olQmV
vlfPZ2+LZl4QTqz5A+DuCfszQ/yeNasbZDDvYaRws+QSHnFbva3GFSvI/g3tgHef
atMdGZT0x3gpqQFzBdhXjU6DOffJY0bOMDlU3/0nDVTwHQjou854miJaECJkSEdX
p5dn+KuaqGqntOam16R032DvFnJkzHWo3eeXKWdqAmNduzMwep6b/cUihdhIPLpS
3wqFvTVPS0Dthj905uUtKkohsz99BjyZtf8kDFZ1b+E1TMkSiJmt76KMwEDCUFE1
4Ix24FRGmMlN9aHVRCeOiQ==</ds:SignatureValue>
    </ds:Signature>
    <saml:Subject>
      <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">user@example.com</saml:NameID>
      <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <saml:SubjectConfirmationData NotOnOrAfter="2024-01-02T00:00:00Z" Recipient="https://sp.example.com/acs" InResponseTo="_request_id"/>
      </saml:SubjectConfirmation>
    </saml:Subject>
    <saml:Conditions NotBefore="2024-01-01T00:00:00Z" NotOnOrAfter="2024-01-02T00:00:00Z">
      <saml:AudienceRestriction>
        <saml:Audience>https://sp.example.com</saml:Audience>
      </saml:AudienceRestriction>
    </saml:Conditions>
    <saml:AttributeStatement>
      <saml:Attribute Name="urn:oid:0.9.2342.19200300.100.1.3" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" FriendlyName="mail">
        <saml:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xs="http://www.w3.org/2001/XMLSchema" xsi:type="xs:string">user@example.com</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="urn:oid:1.3.6.1.4.1.5923.1.1.1.1" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
        <saml:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xs="http://www.w3.org/2001/XMLSchema" xsi:type="xs:string">admin</saml:AttributeValue>
        <saml:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xs="http://www.w3.org/2001/XMLSchema" xsi:type="xs:string">user</saml:AttributeValue>
      </saml:Attribute>
    </saml:AttributeStatement>
  </saml:Assertion>
</samlp:Response>
```

Signature validation happens in two phases.

First, the `<ds:SignedInfo>` block is located, and canonicalized by whatever's
specified in the `<ds:CanonicalizationMethod>`. The output is compared against
the `<ds:SignatureValue>` using an out-of-band provided public key. The
`<ds:SignedInfo>` element is now validated.

Second, the library follows the `<ds:Reference URI="#_assertion_id">` element
within the SignedInfo to determine what element has been signed. Importantly,
either the Response or the Assertion element can be signed (in the above
example, the assertion is). Transforms then occur on that element that remove
the signature, canonicalize the result, and compare it against the
`<ds:DigestValue>`.

If everything matches, the target element is then parsed, and attributes are
pulled out of the assertion.

## Common issues

Almost all SAML bypasses are the result of signature validation passing, but the
XML parsing returning subtly different results. The most straightforward version
of this is taking a valid response, and wrapping it in a malicious document.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<samlp:Response ...>
  <!--Attacker provided elements that the parser reads-->
  <samlp:Extensions ...>
    <!--Fully signed original document hidden from the parser that the XML
    signature validation passes on-->
  </samlp:Extensions>
</samlp:Response>
```

If the signature validation looks for a `<Signature>` block, and validates the
Reference URI, but the parser simply parses the Response, signature validation
will pass, but the parser will read malicious values. Packages can either only
parse the canonicalized bytes (strong prevention), or attempt to detect if XML
nodes point to different points in memory (less strong).

More advanced techniques include:

- Tricking the response URI search using misalignments in different xpath
  expressions to point to attacker provided elements. This can get arbitrarily
  clever.
- Using different canonicalization methods to insert comments or external
  namespace references.
