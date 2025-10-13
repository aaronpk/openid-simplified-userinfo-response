---
title: "OpenID Connect Simplified Userinfo Response"
abbrev: "Simplified Userinfo Response"
category: info

docname: draft-openid-simplified-userinfo-response-latest
submissiontype: independent
number:
date:
v: 3
# area:
workgroup: "OpenID Connect"
keyword:
 - openid
venue:
  group: "OpenID Connect"
  type: Working Group
  mail: openid-specs-ab@lists.openid.net
  github: "aaronpk/openid-simplified-subject-response"
  latest: "https://aaronpk.github.io/openid-simplified-userinfo-response/draft-openid-simplified-userinfo-response.html"

author:
 -
    fullname: Aaron Parecki
    organization: Okta
    email: aaron@parecki.com

normative:
  RFC6749:
  RFC7636:
  OpenID:
    title: OpenID Connect Core 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-core-1_0.html
    date: December 15, 2023
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: B. de Medeiros
      - ins: C. Mortimore

informative:
  RFC9126:

...

--- abstract

This specification defines an extension to the OpenID Connect OAuth Token Endpoint Response to allow a Relying Party (RP) to retrieve the user's info (e.g. subject identifier or profile info) directly in the token response, rather than being encapsulated within a full ID Token.

--- middle

# Introduction

This specification defines an extension to the OpenID Connect [OpenID] OAuth Token Endpoint Response to allow a Relying Party (RP) to retrieve the user's info directly in the token response, rather than being encapsulated within a full ID Token.

This simplifies client implementations that do not require signed statements about the user's authentication from the OpenID Provider, and wish to avoid the overhead, complexity, and dependency management associated with parsing and validating a JSON Web Token (JWT).

This specification provides two different ways the OpenID Provider can return info in the Token Response:

* The "Simplified Subject Response", returning only the `sub` claim in the token response
* The "Simplified Userinfo Response", returning a new top-level property `id_info` as a JSON object containing the same claims that would have been in the ID token.

This mechanism MUST NOT be used as a replacement for a full OIDC flow when the RP requires a signed response, such as when the RP needs non-repudiation of the claims in the ID token.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Simplified Subject Response

To signal its intent to receive only the Subject Identifier in the Token Response, the Relying Party (OAuth Client) uses a new scope value in the Authorization Request.

## Requesting the Simplified Subject Response

This specification defines a new scope value: `subject`

`subject`
: Requests that the Authorization Server (AS) return the Subject Identifier (`sub`) directly in the Token Response, and omit the `id_token`.

## Authorization Request

The Client SHOULD include the `subject` scope along with the standard `openid` scope and any other required scopes in the Authorization Request.

Example request:

    GET /authorize?
      response_type=code
      &client_id=e69b14fe3fdcf1b432deb0c
      &scope=openid%20subject
      &state=9e6d47801c833bfc8
      &redirect_uri=https%3A%2F%2Fclient.example.com%2Fcb
      &code_challenge=dy57vyachQ...&code_challenge_method=S256

Note: This is not limited to the [RFC6749] Authorization Code flow and applies the same to extensions such as Pushed Authorization Request [RFC9126].

## Token Endpoint Response

This specification defines a new field to be returned in the Token Endpoint response: `sub`.

In the Authorization Code flow, upon successful validation of the Authorization Code, the PKCE Code Verifier, and optional client authentication, the Authorization Server constructs a Token Response including the new field `sub`.

The value of the `sub` MUST be identical to the value of the `sub` that would have otherwise been returned in the ID token.

## Response Rules

* If the Client included the `subject` scope in the initial Authorization Request, the OpenID Provider MUST include the `sub` parameter in the Token Response. The OpenID Provider MAY also return the `sub` parameter if the Client did not include the `subject` scope in the initial Authorization Request.
* If the `sub` parameter is included in the Token Response, the Authorization Server MUST NOT include the `id_token` parameter.
* No change is made to the requirements of the standard OAuth 2.0 [RFC6749] parameters in the response or any additional parameters defined by extensions.

Example successful token response

    {
      "access_token": "a5c64fbb3e03d973d3c7ef",
      "token_type": "Bearer",
      "expires_in": 3600,
      "sub": "b46b2f1f2b7686d"
    }

# Simplified Userinfo Response

To signal its intent to receive the user info in the Token Response instead of in an ID token, the Relying Party (OAuth Client) uses a new scope value in the Authorization Request.

## Requesting the Simplified Subject Response

This specification defines a new scope value: `id_info`

`id_info`
: Requests that the Authorization Server (AS) return the user info directly in the Token Response, and omit the `id_token`.

## Authorization Request

The Client SHOULD include the `id_info` scope along with the standard `openid` scope and any other required scopes in the Authorization Request.

Example request:

    GET /authorize?
      response_type=code
      &client_id=e69b14fe3fdcf1b432deb0c
      &scope=openid%20id_info
      &state=9e6d47801c833bfc8
      &redirect_uri=https%3A%2F%2Fclient.example.com%2Fcb
      &code_challenge=dy57vyachQ...&code_challenge_method=S256

Note: This is not limited to the [RFC6749] Authorization Code flow and applies the same to extensions such as Pushed Authorization Request [RFC9126].

## Token Endpoint Response

This specification defines a new field to be returned in the Token Endpoint response: `id_info`.

In the Authorization Code flow, upon successful validation of the Authorization Code, the PKCE Code Verifier, and optional client authentication, the Authorization Server constructs a Token Response including the new field `id_info`.

The value of the `id_info` is a JSON object, and the claims MUST be identical to the claims about the user and their authentication that would have otherwise been returned in the ID token. The following claims about the OpenID Provider and ID token itself SHOULD NOT be returned: `iss`, `aud`, `nonce`.

## Response Rules

* If the Client included the `id_info` scope in the initial Authorization Request, the OpenID Provider MUST include the `id_info` parameter in the Token Response. The OpenID Provider MAY also return the `id_info` parameter if the Client did not include the `id_info` scope in the initial Authorization Request.
* If the `id_info` parameter is included in the Token Response, the Authorization Server MUST NOT include the `id_token` parameter.
* No change is made to the requirements of the standard OAuth 2.0 [RFC6749] parameters in the response or any additional parameters defined by extensions.

Example successful token response

    {
      "access_token": "a5c64fbb3e03d973d3c7ef",
      "token_type": "Bearer",
      "expires_in": 3600,
      "id_info": {
        "sub": "b46b2f1f2b7686d",
        "exp": 1311281970,
        "auth_time": 1311280969,
        "acr": "phr"
      }
    }


# Security Considerations

## Token Endpoint

The Subject Identifier and Userinfo response fields MUST only be returned from the token endpoint.

The Subject Identifier and Userinfo MUST NOT be returned in the Authorization Response (the URL redirect back to the client), as this passes through the front channel and this shortcut provides no mechanism to for the RP to validate the legitimacy of the value in the response.

## Authorization Code Flow

If using the Authorization Code flow, or any flow that utilizes a browser redirect, the flow MUST be secured with PKCE as defined in [RFC7636] to prevent authorization code injection attacks.

## Tamper Resistance

The simplified response defined here provides no cryptographic signature over the `sub` or `id_info` values. Their integrity and authenticity are solely reliant on the security of the transport layer of the Token Endpoint and the preceding Authorization Code + PKCE exchange.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank the following people for their review and contributions to this draft: Brian Campbell.

