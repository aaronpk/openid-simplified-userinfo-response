---
title: "OAuth Userinfo Response"
abbrev: "OAuth Userinfo Response"
category: info

docname: draft-parecki-oauth-userinfo-response-latest
submissiontype: IETF
number:
date:
v: 3
area: "Security"
workgroup: "Web Authorization Protocol"
keyword:
 - oauth
 - userinfo
venue:
  group: "Web Authorization Protocol"
  type: Working Group
  mail: oauth@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/oauth/
  github: "aaronpk/oauth-userinfo-response"
  latest: "https://drafts.aaronpk.com/oauth-userinfo-response/draft-parecki-oauth-userinfo-response.html"

author:
 -
    fullname: Aaron Parecki
    organization: Okta
    email: aaron@parecki.com

normative:
  RFC6749:
  RFC7519:
  RFC7636:

informative:
  RFC9126:

...

--- abstract

This specification defines an extension to the OAuth 2.0 Token Endpoint Response to allow the authorization server to return an identifier for the user who authorized the request, and optionally additional information about that user, directly in the token response as plain JSON.

--- middle

# Introduction

This specification defines an extension to the OAuth 2.0 [RFC6749] Token Endpoint Response to allow the authorization server to return an identifier for the user who authorized the request, and optionally additional information about that user, directly in the token response.

OAuth 2.0 does not define a way for the authorization server to tell the client which user authorized the request. Many clients only need to know a user identifier, for example to associate the authorization with a local account, and do not require a signed statement about the user or their authentication from the authorization server. Returning this information as plain JSON in the token response avoids an additional request, as well as the overhead, complexity, and dependency management associated with parsing and validating a JSON Web Token (JWT).

This specification provides two different ways the authorization server can return info in the Token Response:

* The "Subject Response", returning only the user identifier as a new top-level property `sub` in the token response
* The "Userinfo Response", returning a new top-level property `userinfo` as a JSON object containing the user identifier and any additional claims the authorization server chooses to return.

This specification only gives the authorization server a way to return a user identifier to the client. It does not prove the identity of the user, and the identifier says nothing about a real person: it is the authorization server's own identifier for the account that authorized the request.

This mechanism MUST NOT be used when the client requires a signed statement about the user, such as when the client needs non-repudiation of the claims.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This specification uses the terms "authorization server", "client", "resource owner", "authorization request", "token endpoint", and "scope" defined by OAuth 2.0 [RFC6749]. The term "user" refers to the resource owner who authorized the request.


# Subject Response

To signal its intent to receive the user identifier in the Token Response, the client uses a new scope value in the Authorization Request.

## Requesting the Subject Response

This specification defines a new scope value: `subject`

`subject`
: Requests that the authorization server return the user identifier (`sub`) directly in the Token Response.

## Authorization Request

The client SHOULD include the `subject` scope along with any other required scopes in the Authorization Request.

Example request:

    GET /authorize?
      response_type=code
      &client_id=e69b14fe3fdcf1b432deb0c
      &scope=subject
      &state=9e6d47801c833bfc8
      &redirect_uri=https%3A%2F%2Fclient.example.com%2Fcb
      &code_challenge=dy57vyachQ...&code_challenge_method=S256

Note: This is not limited to the [RFC6749] Authorization Code flow and applies the same to extensions such as Pushed Authorization Request [RFC9126].

## Token Endpoint Response {#subject-token-response}

This specification defines a new field to be returned in the Token Endpoint response: `sub`.

In the Authorization Code flow, upon successful validation of the Authorization Code, the PKCE Code Verifier, and optional client authentication, the authorization server constructs a Token Response including the new field `sub`.

The value of `sub` is a string containing the authorization server's identifier for the user who authorized the request. It has the same meaning as the `sub` claim defined in {{Section 4.1.2 of RFC7519}}. The value MUST be unique within the authorization server and MUST NOT be reassigned to a different user. The value is an opaque identifier that is only meaningful in the context of the authorization server that issued it.

## Response Rules

* If the client included the `subject` scope in the initial Authorization Request, the authorization server MUST include the `sub` parameter in the Token Response. The authorization server MAY also return the `sub` parameter if the client did not include the `subject` scope in the initial Authorization Request.
* No change is made to the requirements of the standard OAuth 2.0 [RFC6749] parameters in the response or any additional parameters defined by extensions.

Example successful token response

    {
      "access_token": "a5c64fbb3e03d973d3c7ef",
      "token_type": "Bearer",
      "expires_in": 3600,
      "sub": "b46b2f1f2b7686d"
    }

# Userinfo Response

To signal its intent to receive information about the user in the Token Response, the client uses a new scope value in the Authorization Request.

## Requesting the Userinfo Response

This specification defines a new scope value: `userinfo`

`userinfo`
: Requests that the authorization server return information about the user directly in the Token Response.

## Authorization Request

The client SHOULD include the `userinfo` scope along with any other required scopes in the Authorization Request.

Example request:

    GET /authorize?
      response_type=code
      &client_id=e69b14fe3fdcf1b432deb0c
      &scope=userinfo
      &state=9e6d47801c833bfc8
      &redirect_uri=https%3A%2F%2Fclient.example.com%2Fcb
      &code_challenge=dy57vyachQ...&code_challenge_method=S256

Note: This is not limited to the [RFC6749] Authorization Code flow and applies the same to extensions such as Pushed Authorization Request [RFC9126].

## Token Endpoint Response {#userinfo-token-response}

This specification defines a new field to be returned in the Token Endpoint response: `userinfo`.

In the Authorization Code flow, upon successful validation of the Authorization Code, the PKCE Code Verifier, and optional client authentication, the authorization server constructs a Token Response including the new field `userinfo`.

The value of `userinfo` is a JSON object containing the user identifier and any additional claims the authorization server chooses to return. The claim names and their meanings are those registered in the IANA "JSON Web Token Claims" registry established by [RFC7519]. The object MUST contain the `sub` claim, as described in {{subject-token-response}}. Which additional claims are returned is at the discretion of the authorization server, and may depend on the scopes granted.

## Response Rules

* If the client included the `userinfo` scope in the initial Authorization Request, the authorization server MUST include the `userinfo` parameter in the Token Response. The authorization server MAY also return the `userinfo` parameter if the client did not include the `userinfo` scope in the initial Authorization Request.
* No change is made to the requirements of the standard OAuth 2.0 [RFC6749] parameters in the response or any additional parameters defined by extensions.

Example successful token response

    {
      "access_token": "a5c64fbb3e03d973d3c7ef",
      "token_type": "Bearer",
      "expires_in": 3600,
      "userinfo": {
        "sub": "b46b2f1f2b7686d",
        "name": "Alice Example",
        "preferred_username": "alice",
        "picture": "https://as.example.com/u/b46b2f1f2b7686d.jpg"
      }
    }


# Security Considerations

## Not a Proof of Identity

The `sub` and `userinfo` values are not an assertion of the user's identity. They tell the client which identifier the authorization server associates with the user who authorized the request, and nothing more. In particular, they do not establish that the user is a particular real person. Clients MUST treat the user identifier as scoped to the authorization server that returned it, and MUST NOT assume that the same value returned by a different authorization server refers to the same user.

## Token Endpoint

The `sub` and `userinfo` response fields MUST only be returned from the token endpoint.

The `sub` and `userinfo` values MUST NOT be returned in the Authorization Response (the URL redirect back to the client), as this passes through the front channel and provides no mechanism for the client to validate the legitimacy of the value in the response.

## Authorization Code Flow

If using the Authorization Code flow, or any flow that utilizes a browser redirect, the flow MUST be secured with PKCE as defined in [RFC7636] to prevent authorization code injection attacks.

## Tamper Resistance

The response defined here provides no cryptographic signature over the `sub` or `userinfo` values. Their integrity and authenticity are solely reliant on the security of the transport layer of the Token Endpoint and the preceding Authorization Code + PKCE exchange.


# IANA Considerations

## OAuth Parameters Registration

IANA is requested to register the following values in the "OAuth Parameters" registry of the "OAuth Parameters" registry group established by {{Section 11.2 of RFC6749}}.

Name:
: `sub`

Parameter Usage Location:
: token response

Change Controller:
: IETF

Reference:
: {{subject-token-response}} of this document

Name:
: `userinfo`

Parameter Usage Location:
: token response

Change Controller:
: IETF

Reference:
: {{userinfo-token-response}} of this document


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank the following people for their review and contributions to this draft: Brian Campbell.
