---
layout: post
title:  "On the Importance of Trust Validation: Microsoft's Dangerous Mistake"
image: '2020-08-26-bg'
date:   2020-08-26 00:00:00 +0000
author: Daniel Ostovary
summary: "Our discovery of how Microsoft didn't verify the validity of timestamping certificates on VSIX packages"
description:
---

Last year we discovered a vulnerability in the Visual Studio Extension (VSIX) installer, which comes with Microsoft Visual Studio. When verifying the signature of a VSIX package, the VSIX installer failed to check the trust of the timestamp under certain circumstances. This vulnerability allowed an attacker to apply a non-trustworthy timestamp without users being warned about it. 

In this blog post we are talking about this vulnerability and its impact. The blog post is written in cooperation with Marc Nimmerichter from [Impidio](https://www.impidio.com/blog/on-the-importance-of-trust-validation-microsofts-dangerous-mistake). The here-discussed vulnerability has been reported to Microsoft shortly after its discovery. We will talk about our experiences with reporting this vulnerability in a later blog post. To understand the vulnerability, one has to understand timestamping in code signing first.

# Timestamping
Unlike signatures of TLS certificates, signatures of code-signing certificates may be verified a long time after their application (e.g. a software publisher may apply a signature on an .EXE file in 2018; a user may use this .EXE in 2024). Code-signing certificates have a relatively low validity period of 1-3 years [[1]](#1). As a result, upon execution of a program, a user may be presented with a certificate expiration warning because the code-signing certificate is no longer valid at the time the signed program is executed [[2]](#2). Simply giving a code-signing certificate a long validity is considered bad practice. Instead, in code-signing, so-called timestamps are frequently used [[1]](#1). These timestamps are provided by trustworthy Timestamping Authorities (TSAs) and cryptographically guarantee that certain data existed at a certain time [[2]](#2). This means, when applying a timestamp to a signature, it is cryptograhically guaranteed by the issuing TSA that the signature was applied at the time of the timestamp [[2]](#2).

With such a timestamp applied to a signature, **the validity period of a code signature is extended** to the validity period of the TSA's certificate [[4]](#4). The validity period of a TSA's certificate is usually relatively long (typically 10-15 years; e.g. see [[5-7]](#5)). 

Timestamping can also be used to **keep signatures of a revoked code-signing certificate valid if and only if (iff) the revocation date is after the timestamp date**, as suggested by RFC 3161 [[8]](#8).

It is important to check the correctness and trustworthiness of a timestamp [[3]](#3). Otherwise it may be possible to maliciously alter the validity of a code signature.

# The Vulnerability
In the case of the VSIX installer, an attacker was able to apply a valid code signature to a file using an expired code-signing certificate and a self-created non-trustworthy TSA. For that, the attacker had to apply a code signature using the expired code-signing ceritificate, backdating the signature to a time when the code-signing certificate was valid, and apply a specially crafted timestamp. This timestamp would have to:

 1. contain the certificate chain of the self-created non-trustworthy TSA
 2. be dated before the expiration of the code-signing certificate

Before the vulnerability was fixed, the VSIX installer accepted such a combination of a signature and timestamp, i.e. it did not report a non-trustworthy signature/timestamp. This behavior shows that the VSIX installer did not correctly check the trustworthiness of a TSA.

# Impact
We discovered three potential attack scenarios for this vulnerability. One of which is infeasible due to the way the VSIX installer supports timestamping.

**Scenario 1 ('reviving' expired code-signing certificates):** The vulnerability allows an attacker to craft malicious VSIX packages with 'revived' expired code-signing certificates, i.e. **use expired code-signing certificates to sign VSIX packages**. This is problematic because owners of certificates may not protect expired certificates anymore or may not revoke expired code-signing certificates in case of theft because these certificates should be unusable anyway. Attackers can abuse such certificate 'revival' to sign malicious code, which would be accepted by the VSIX installer without any warning and subsequently executed by unsuspecting end users. Essentially, in this scenario an attacker would maliciously **extend the validity period of a code-signing certificate**. There are no specific code-signing certificates for VSIX packages. So any code-signing certificate can be used to sign VSIX packages and subsequently any expired code-signing certificate could be used for this attack.

Imagine the following example: On February 20th 2020 an attacker steals a code-signing certificate that expired on January 10th from a software publisher. The software publisher has not detected the theft or has detected the theft but decided to not revoke the certificate, because the theft was after the expiration of the certificate. The attacker now wants to 'revive' the certificate. For that, the attacker dates back the system time to January 9th 2020 and signs the VISX package with the expired certificate. Then the attacker timestamps the signature for January 9th 2020 with their self-created TSA, using a timestamp that contains the timestamping certificate. After that, the attacker distributes the VISX package to end users (e.g. by publishing it on a website, by performing Man-in-the-Middle (MitM) attacks on a file download, or by putting it in a shared directory). When an end user executes the VSIX package, the signature and timestamp are shown as valid and the VSIX package is subsequently installed without any warning.

**Scenario 2 (applying short-lived timestamps):** The vulnerability further allows an attacker that can intercept and modify HTTP requests/responses (i.e. the attacker is MiTM) **to respond to a software publisher's legitimate timestamping request with a timestamp with a very short lifespan**, without the software publisher being warned. As soon as the software publisher's code-signing certificate expires, the signature would become invalid unexpectedly. This could heavily diminish the usage of the victim's VSIX package, depending on the number of new installations. Timestamp requests are often sent via HTTP (see [[9]](#9)), so Man-in-the-Middle attacks are not unlikely.

Imagine the following example: An attacker is MiTM in the communication between a software publisher and their TSA. The attacker has set up their own non-trustworthy TSA. Today is July 15th 2020. The code-signing certificate of the software publisher expires on July 20th 2020. The certificate of the TSA that the software developers trusts (i.e. the timestamping certificate) expires on January 1st 2030. To extend the validity period of a signature, the software publisher timestamps their signature with the TSA they trust. The attacker intercepts this call and responds with a timestamp that contains the timestamping certificate of their own non-trustworthy TSA. After signing and timestamping their VSIX package, the software publisher tests if the signing/timestamping worked properly by installing the VSIX package with the VSIX installer. The VSIX installer will show a valid signature and timestamp without any indication of the validity period of the signature or the timestamp. Consecutively, the software publisher starts publishing their software as usual on July 18th 2020. In the first days several hundred users download the VSIX package. On July 21st 2020, first users report that the signature on the VSIX package is invalid because the underlying certificate is expired. Now the software publisher has to quickly publish the VSIX package with a new code-signing certificate. 

**Scenario 3 (dodged a bullet - 'reviving' revoked code-signing certificates):** Our tests have shown that the VSIX installer only checks if a code-signing certificate was revoked, but not if it was timestamped before the revocation date. This means, the VSIX installer rejects all revoked certificates regardless of the timestamp (i.e. it does **NOT keep signatures of revoked code-signing certificates valid iff the revocation date is after the timestamp date**). As a result of the VSIX installer's behavior, this vulnerability did not allow to 'revive' revoked code-signing certificates. If the VSIX installer allowed revoked code-signing certificates to stay valid iff the revocation date is after the timestamp date, an attacker could have **'revived' revoked code-signing certificates** (i.e. make revoked code-signing certificates valid again). This would have worked similar to the example of scenario 1. Instead of backdating the signature and timestamp to a date when the certificate was expired, an attacker would have backdated the signature and timestamp to a date when the certificate was not revoked. Needless to say that the possibility of 'reviving' revoked code-signing certificates would have constituted a critical vulnerability. Essentially, Microsoft unintentionally dodged a huge bullet by NOT **keeping signatures of revoked code-signing certificates valid iff the revocation date is after the timestamp date**.

# Security Patch
Microsoft has informed us that they were planning to fix the vulnerability with Visual Studio 16.3, which was released in Fall 2019. Old versions of Visual Studio will not be patched, and thus will remain vulnerable indefinitely. Unfortunately, the release notes of Visual Studio 16.3. did not mention the here-described vulnerability in any way [[10]](#10). However, in May 2020 we could confirm that the vulnerability is fixed at least in versions >=16.5.2047. 

<div class='sources' markdown='1'>
# Sources
* \[<span id='1'>1</span>\] [https://knowledge.digicert.com/generalinformation/INFO1119.html](https://knowledge.digicert.com/generalinformation/INFO1119.html)
* \[<span id='2'>2</span>\] [https://casecurity.org/wp-content/uploads/2013/10/CASC-Code-Signing.pdf](https://casecurity.org/wp-content/uploads/2013/10/CASC-Code-Signing.pdf)
* \[<span id='3'>3</span>\] [https://csrc.nist.gov/CSRC/media/Publications/white-paper/2018/01/26/security-considerations-for-code-signing/final/documents/security-considerations-for-code-signing.pdf](https://csrc.nist.gov/CSRC/media/Publications/white-paper/2018/01/26/security-considerations-for-code-signing/final/documents/security-considerations-for-code-signing.pdf)
* \[<span id='4'>4</span>\] [https://www.xolphin.com/support/signatures/Frequently_asked_questions/Timestamping](https://www.xolphin.com/support/signatures/Frequently_asked_questions/Timestamping)
* \[<span id='5'>5</span>\] [https://support.sectigo.com/ES_KnowledgeDetailPageFaq?Id=kA01N000000btid](https://support.sectigo.com/ES_KnowledgeDetailPageFaq?Id=kA01N000000btid)
* \[<span id='6'>6</span>\] [https://www.sede.fnmt.gob.es/documents/10445900/10577712/dpstq_english.pdf](https://www.sede.fnmt.gob.es/documents/10445900/10577712/dpstq_english.pdf)
* \[<span id='7'>7</span>\] [https://www.digicert-grid.com/DigiCert_CP_v403.pdf](https://www.digicert-grid.com/DigiCert_CP_v403.pdf)
* \[<span id='8'>8</span>\] [https://www.ietf.org/rfc/rfc3161.txt](https://www.ietf.org/rfc/rfc3161.txt)
* \[<span id='9'>9</span>\] [https://gist.github.com/Manouchehri/fd754e402d98430243455713efada710](https://gist.github.com/Manouchehri/fd754e402d98430243455713efada710)
* \[<span id='10'>10</span>\] [https://docs.microsoft.com/en-us/visualstudio/releases/2019/release-notes-v16.3](https://docs.microsoft.com/en-us/visualstudio/releases/2019/release-notes-v16.3)
</div>
