---
title: "ACME IP Identifier Validation Extension"
abbrev: ACME-IP
docname: draft-ietf-acme-ip-latest
category: std

ipr: trust200902
area: General
workgroup: ACME Working Group

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: R. B. Shoemaker
    name: Roland Bracewell Shoemaker
    org: ISRG
    email: roland@letsencrypt.org

normative:
  RFC1034:
  RFC2119:
  I-D.ietf-acme-acme:

--- abstract

The Automatic Certificate Management Environment (ACME) protocol only defines
identity validation challenges for DNS host names. This document provides
guidance on valiadtion of IPv4 and IPv6 addresses which can then be included
in X.509 certificates.

--- middle

# Introduction

The identity validation challenges defined in ACME {{ID.ietf-acme-acme}} only
apply to validation of DNS host names. In order to allow validation of IPv4
and IPv6 addresses for inclusion in X.509 certificates this document defines
a new challenge and provides guidance on how already defined challenges can
be used with these identifiers.

# Terminology

In this document, the key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" are to be
interpreted as described in BCP 14, RFC 2119 {{RFC2119}} and indicate
requirement levels for compliant ACME-Wildcard implementations.

# IP Identifier

ACME {{ID.ietf-acme-acme}} only defines the identifier type "dns" which is used
to refer to fully qualified domain names. If a ACME server wishes to request
proof that a user controls a IPv4 or IPv6 identifier it may create an
authorization with the type "ip". The value field of the identifier must contain
the dot decimal notation format of the address as defined in RFC 3986 Section
3.2.2.

# Identifier Validation Challenges

When creating an authorization for a IP identifier the following challenge types
may be used to perform validation.

## Reverse DNS

With Reverse DNS validation the client proves control of an IP address by
provisioning a TXT resource record containing a designated value for a
specific validation domain name.

type (required, string):
: The string "reverse-dns-01"

token (required, string):
: A random value that uniquely identifies the challenge.  This value MUST have
at least 128 bits of entropy, in order to prevent an attacker from guessing it.
It MUST NOT contain any characters outside the base64url alphabet, including
padding characters ("=").

~~~~~~~~~~
GET /acme/authz/1234/2 HTTP/1.1
Host: example.com

HTTP/1.1 200 OK
{
  "type": "reverse-dns-01",
  "url": "https://example.com/acme/authz/1234/2",
  "status": "pending",
  "token": "evaGxfADs6pSRb2LAv9IZf17Dt3juxGJ-PCt92wr-oA"
}
~~~~~~~~~~

A client responds to this challenge by constructing a key authorization from the
"token" value provided in the challenge and the client's account key.  The
client then computes the SHA-256 digest [FIPS180-4] of the key authorization.

The record provisioned to the DNS is the base64url encoding of this digest. The
client constructs the validation domain name by prepending the label
"_acme-challenge" to the domain name referenced in the PTR resource record for
the IN-ADDR.ARPA {{!RFC1034}} mapping of the IP address. The client then
provisions a TXT record with the digest for this name. For example, if the IP
address being validated is "192.0.2.1" and its IN-ADDR.ARPA mapping had the
following PTR record:

~~~~~~~~~~
1.2.0.192.in-addr.arpa. 300 IN PTR example.com
~~~~~~~~~~

then the client would provision the following DNS record:

~~~~~~~~~~
_acme-challenge.example.com. 300 IN TXT "gfj9Xq...Rg85nM"
~~~~~~~~~~

The response to the Reverse DNS challenge provides the computed key authorization
to acknowledge that the client is ready to fulfill this challenge.

keyAuthorization (required, string):
: The key authorization for this challenge.  This value MUST match the token
from the challenge and the client's account key.

~~~~~~~~~~
POST /acme/authz/1234/2
Host: example.com
Content-Type: application/jose+json

{
  "protected": base64url({
    "alg": "ES256",
    "kid": "https://example.com/acme/acct/1",
    "nonce": "JHb54aT_KTXBWQOzGYkt9A",
    "url": "https://example.com/acme/authz/1234/2"
  }),
  "payload": base64url({
    "keyAuthorization": "evaGxfADs...62jcerQ"
  }),
  "signature": "Q1bURgJoEslbD1c5...3pYdSMLio57mQNN4"
}
~~~~~~~~~~

On receiving a response, the server MUST verify that the key authorization in
the response matches the "token" value in the challenge and the client's account
key.  If they do not match, then the server MUST return an HTTP error in
response to the POST request in which the client sent the challenge.

To validate a DNS challenge, the server performs the following steps:

1. Compute the SHA-256 digest [FIPS180-4] of the key authorization
2. Query for PTR records for the IP identifiers IN-ADDR.ARPA mapping
2. Query for TXT records for the validation domain name
3. Verify that the contents of one of the TXT records matches the digest value

If all of the above verifications succeed, then the validation is successful.
If no DNS records are found, or DNS records and response payload do not pass these
checks, then the validation fails.

## Existing Challenges

IP identifiers can be used with the existing HTTP and TLS-SNI challenges from
RFC XXX Section XXX by skipping the DNS resolution step and using the relevant
IP address as the HTTP or TLS server to directly connect to.

# IANA Considerations

## Identifier Types

Adds a new type to the Identifier list defined in Section XXX of RFC XXXX with
the label 'ip' with the reference RFC XXXX.

## Challenge Types

Adds a new type to the Challenge list defined in Section XXX of RFC XXXX with
the label 'reverse-dns', identifier type 'ip', and reference RFC XXXX.

# Security Considerations

## Certificate Lifetime

Given the often short assignment periods of IP addresses provided by various
service providers CAs MAY want to impose much shorter lifetimes for certificates
which contain IP identifiers. They MAY also impose restrictions on IP
identifiers which are in CIDRs which are assigned to service providers which are
known to dynamically assign addresses from these CIDRs to users for
indeterminate periods of time.
