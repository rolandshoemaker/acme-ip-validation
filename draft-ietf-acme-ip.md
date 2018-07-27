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
    org: Internet Security Research Group
    abbrev: ISRG
    email: roland@letsencrypt.org

normative:
  RFC1034:
  RFC1123:
  RFC2119:
  RFC3596:
  RFC4291:
  RFC4648:
  RFC6066:
  RFC7230:
  I-D.ietf-acme-acme:
  I-D.ietf-acme-tls-alpn:
  FIPS180-4:
    title: NIST FIPS 180-4, Secure Hash Standard
    author:
      name: NIST
      ins: National Institute of Standards and Technology, U.S. Department of Commerce
    date: 2012-03
    target: http://csrc.nist.gov/publications/fips/fips180-4/fips-180-4.pdf

--- abstract

This document specifies identifiers and challenges required to enable the Automated Certificate Management Environment (ACME) to issue certificates for IP addresses.

--- middle

# Introduction

The Automatic Certificate Management Environment (ACME) {{I-D.ietf-acme-acme}} only defines challenges for validating control of DNS host name identifiers which limits its use to being used for issuing certificates for DNS identifiers. In order to allow validation of IPv4 and IPv6 identifiers for inclusion in X.509 certificates this document specifies how challenges defined in the original ACME specification and the TLS-ALPN extension specification {{I-D.ietf-acme-tls-alpn}} can be used to validate IP identifiers.

# Terminology

In this document, the key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in BCP 14, {{RFC2119}}.

# IP Identifier

{{I-D.ietf-acme-acme}} only defines the identifier type "dns" which is used to refer to fully qualified domain names. If a ACME server wishes to request proof that a user controls a IPv4 or IPv6 address it MUST create an authorization with the identifier type "ip". The value field of the identifier MUST contain the textual form of the address as defined in {{RFC1123}} Section 2.1 for IPv4 and in {{RFC4291}} Section 2.2 for IPv6.

An identifier for the IPv6 address 2001:db8::1 would be formatted like so:

~~~~~~~~~~
{"type": "ip", "value": "2001:db8::1"}
~~~~~~~~~~

# Identifier Validation Challenges

IP identifiers MAY be used with the existing "http-01" and "tls-alpn-01" challenges from {{I-D.ietf-acme-acme}} Section 8.3 and {{I-D.ietf-acme-tls-alpn}} Section 3 respectively. To use IP identifiers with these challenges their initial DNS resolution step MUST be skipped and the IP address used for validation MUST be the value of the identifier.

For the "http-01" challenge the Host header MUST be set to the IP address being used for validation per {{RFC7230}}.

For the "tls-alpn-01" the subjectAltName extension in the validation certificate MUST contain a single iPAddress which matches the address being validated. As {{RFC6066}} does not permit IP addresses to be used in the SNI extension the server MUST instead use the IN-ADDR.ARPA {{RFC1034}} or IP6.ARPA {{RFC3596}} reverse mapping of the IP address as the SNI value instead of the literal IP address.

The existing "dns-01" challenge MUST NOT be used to validate IP identifiers.

# IANA Considerations

## Identifier Types

Adds a new type to the Identifier list defined in Section 9.7.7 of {{I-D.ietf-acme-acme}} with the label "ip" and reference I-D.ietf-acme-ip.

## Challenge Types

Adds the value "ip" to the Identifier Type column in the Validation Methods list defined in Section 9.7.8 of {{I-D.ietf-acme-acme}} for the "http-01" and "tls-alpn-01" challenges.

# Security Considerations

Given the often short delegation periods for IP addresses provided by various service providers CAs MAY want to impose shorter lifetimes for certificates which contain IP identifiers. They MAY also impose restrictions on IP identifiers which are in CIDRs known to be assigned to service providers who dynamically assign addresses to users for indeterminate periods of time.

# Acknowledgments

The author would like to thank those who contributed to this document and offered editorial and technical input, especially Jacob Hoffman-Andrews and Daniel McCarney.
