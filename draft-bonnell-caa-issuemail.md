---
title: "Certification Authority Authorization (CAA) Processing for Email Addresses"
abbrev: "CAA for Email Addresses"
category: std

docname: draft-bonnell-caa-issuemail-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "CBonnell/caa-issuemail"
  latest: "https://CBonnell.github.io/caa-issuemail/draft-bonnell-caa-issuemail.html"

author:
 -
    fullname: Corey Bonnell
    organization: DigiCert, Inc.
    email: corey.bonnell@digicert.com

normative:
  RFC5322:
  RFC5234:
  RFC5891:
  RFC8659:

informative:
  RFC5890:


--- abstract

The Certificate Authority Authorization (CAA) DNS resource record type
provides a mechanism for domains to express the allowed set of
Certificate Authorities that may issue certificates for the domain.
The core CAA specification ([RFC8659]) solely defines Property Tags that
restrict the issuance of certificates that certify domain names; it does
not define a mechanism for domains to restrict the issuance of
certificates that include email addresses. This specification defines a
Property Tag that grants authorization to Certificate Authorities to
issue certificates which certify email addresses.


--- middle

# Introduction

This document defines a CAA Property Tag which restricts the allowed set
of issuers for electronic email addresses. Its syntax and processing
are similar to the "issue" Property Tag as defined in section 4.2 of
[RFC8659].

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The "issuemail" Property Tag Syntax

This document defines the "issuemail" Property Tag. The presence of
one or more "issuemail" Properties in the Relevant Resource Record
Set ([RFC8659]) indicates that the domain is requesting that
Certification Authorities restrict the issuance of certificates that
certify email addresses.

The CAA "issuemail" Property Value has the following sub-syntax
(specified in ABNF as per [RFC5234]):

~~~
  issuemail-value = *WSP [issuer-domain-name *WSP]
    [";" *WSP [parameters *WSP]]

  issuer-domain-name = label *("." label)
  label = (ALPHA / DIGIT) *( *("-") (ALPHA / DIGIT))

  parameters = (parameter *WSP ";" *WSP parameters) / parameter
  parameter = tag *WSP "=" *WSP value
  tag = (ALPHA / DIGIT) *( *("-") (ALPHA / DIGIT))
  value = *(%x21-3A / %x3C-7E)
~~~

Readers who are familiar with the sub-syntax of the "issue" and
"issuewild" Property Tags will recognize that this sub-syntax is
identical.


# "issuemail" Property Tag Processing

Prior to issuing a certificate that certifies an email address, the
Certification Authority MUST check for publication of a Relevant
Resource Record Set (RRSet). The discovery of such a Relevant RRSet MUST
be performed using the algorithm specified in section 3 of [RFC8659].
The input domain to the discovery algorithm SHALL be the domain "part"
([RFC5322]) of the email address that is being certified. If the domain
"part" of the email address being certified is an Internationalized
Domain Name ([RFC5890]) that contains one or more U-Labels, then all
U-Labels MUST be converted to their A-Label representation ([RFC5891])
for the purpose of discovering the Relevant RRSet for that email
address.

If the Relevant RRSet is empty, or the Relevant RRSet does not contain
any "issuemail" Properties , then the domain has not requested any
restrictions on the issuance of certificates for email addresses. The
presence of other Property Tags, such as "issue" or "issuewild", does
not restrict the issuance of certificates which certify email addresses.

For each "issuemail" Property in the Relevant RRSet, the
Certification Authority SHALL compare its issuer-domain-name with the
issuer-domain-name as expressed in the Property Value. If there is not
any "issuemail" record whose issuer-domain-name (as expressed in the
Property Value) matches the Certification Authority's
issuer-domain-name, then the Certification Authority MUST NOT issue
the certificate. If the Relevant RRSet contains any "issuemail"
Property whose issuemail-value does not conform to the ABNF syntax as
defined section 3 of this document, then those records SHALL be treated
as if the issuer-domain-name in the issuemail-value is the empty string.

If the certificate certifies more than one email address, then the
Certification Authority MUST perform the above procedure for each
email address being certified.

The assignment of issuer-domain-names to Certification Authorities is
beyond the scope of this document.

The processing of parameters in the issuemail-value are beyond the scope
of this document.


# "issuemail" Property Tag Examples

Several illustrative examples of Relevant RRSets and their expected
processing semantics follow. All examples assume that the
issuer-domain-name for the Certification Authority is "ca.example.com".

The following RRSet does not contain any "issuemail" Properties,
so there are no restrictions on the issuance of certificates which
certify email addresses for that domain:

~~~
mail.example.com         CAA 0 issue "ca1.example.net"
mail.example.com         CAA 0 issue "ca2.example.org"
~~~

The following RRSet contains a single "issuemail" Property where the
issuer-domain-name is the empty string, so the issuance of certificates
certifying email addresses for the domain is prohibited:

~~~
mail.example.com         CAA 0 issuemail ";"
~~~

The following RRSet contains multiple "issuemail" Properties,
one of which matches the issuer-domain-name of the example Certification
Authority ("ca.example.com") and one Property which does not match.
Given that there is at least one record whose issuer-domain-name
matches the Certification Authority's issuer-domain-name, issuance is
permitted.

~~~
mail.example.com         CAA 0 issuemail ";"
mail.example.com         CAA 0 issuemail "ca.example.com"
~~~

The following RRSet contains a single "issuemail" Property whose
sub-syntax does not conform to the ABNF as specified in section 3. Given
that "issuemail" Properties with malformed syntax are treated the
same as "issuemail" Properties whose issuer-domain-name is the empty
string, issuance is prohibited.

~~~
malformed.example.com     CAA 0 issuemail "%%%%%"
~~~

# Security Considerations

TODO: verify that this is correct

This document introduces no security considerations beyond those
expressed in [RFC8659].

# IANA Considerations

The author(s) request the registration of the following "Certification
Authority Restriction Properties":

| Tag       | Meaning                              | Reference       |
| --------- | ------------------------------------ | --------------- |
| issuemail | Authorization Entry by Email Address | [This document] |


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
