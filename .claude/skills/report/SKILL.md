---
name: report
description: "Generates a report from a finding"
---

For a given finding, write up a report. This should be as concise as possible.

The format should be:

> # {Title}
>
> An extremely brief summary of the attack (a sentence).
>
> A paragraph extremely brief explination what exported function is impacted.
> What internal code causes this issue. Be as high-level as possible. Do NOT
> link to code or go into details.
>
> An example pseudo-document:
>
> ```xml
> <samlp:Response>
>   <samlp:Extension>
>     <samlp:Response>
>       <!--original validly signed document-->
>     </samlp:Response>
>   </samlp:Extension>
>   <saml:Assertion><!--Malicious content--></saml:Assertion>
> </samlp:Response>
> ```

In addition, generate a standalone proof-of-concept bash script demonstrating
this finding. The script should:

- Create a temp directory
- Generate test documents with `openssl` and `xmlsec1`
- Write a test program that:
  - Generates a malicious document from a validily signed one
  - Sets up the library to validate SAML responses
  - Performs SAML validation on the malicious document through the program and
    prints the bad results
- Setup the environment in that directory importing the impacted version of the
  library (pnpm install, go get, etc.)

The test script should be as bear bones and minimal as possible. Not code
comments.
