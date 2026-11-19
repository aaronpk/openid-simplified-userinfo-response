# Existing Examples: User Identifiers in OAuth Token Responses

A catalog of specifications and vendor implementations that return a user identifier or user information in the OAuth token endpoint response, collected as prior art for the "OAuth Userinfo Response" draft.

Three published specifications and at least 28 vendor OAuth 2.0 token endpoints already return a plain user identifier next to `access_token`, and ten more vendors return a profile object there. **IndieAuth (`me`, `profile`) and the AT Protocol OAuth profile (`sub`) are the closest specification precedents**, and draft-ietf-mailmaint-oauth-public required a `username` member in versions -06 and -07 before removing it in -08 on 18 November 2026. No OAuth 2.0 RFC defines such a member: every standards-body answer so far has been a signed assertion (`id_token`), and GNAP (RFC 9635) is the only RFC that returns a plain subject identifier alongside an access token. Among vendors there is no naming convergence: `user_id` is the most common name, the `openid` / `open_id` / `openId` family dominates Chinese platforms, and **only AT Protocol uses `sub`**. Values are split between JSON strings and JSON numbers, and Figma deprecated its numeric member because the numbers overflow common number types. Explicit security guidance is rare; Weibo, AT Protocol, IndieAuth and GNAP are the sources worth quoting. In the IANA OAuth Parameters registry `sub` is registered only for "authorization request", `username` only for "token request", and `userinfo` is not registered at all.

All findings were collected on 2026-11-19. Every URL below comes from the research notes; nothing was re-fetched while writing this catalog.

## How to read the catalog

**Kind** uses three labels: **ID** is a user identifier, **Profile** is user profile data, and **Context** is a non-user identifier (tenant, merchant, patient, workspace, site).

**Evidence** states how each entry was confirmed. "Docs" means the vendor's own documentation page was read. Weaker bases are named in the entry: "SDK source" (vendor-owned client or server code, no documentation page read), "summarising fetch" (the page was read through a tool that summarises, so wording and whitespace are not byte-exact), "archived copy", and "no example" (member documented in a table or schema, but no verbatim response was captured). Entries that could not be confirmed from an official or vendor-owned source appear only in the final "Unconfirmed or not checked" section.

## Specifications: three define a plain user identifier, and one withdrew it

| Specification | Status | Member(s) | Kind | Condition | Evidence |
|---|---|---|---|---|---|
| [IndieAuth](https://indieauth.spec.indieweb.org/#access-token-response) | IndieWeb Living Standard, 11 July 2024 | `me`, `profile` | ID (URL); Profile | `me` required; `profile` optional, requested with `profile` and `email` scopes | Spec text, verbatim examples |
| [AT Protocol OAuth profile](https://atproto.com/specs/oauth#identity-authentication) | atproto specification, living and unversioned | `sub` | ID (account DID) | Required for every session; triggered by the `atproto` scope | Spec text; the spec has no example response |
| [draft-ietf-mailmaint-oauth-public-07](https://www.ietf.org/archive/id/draft-ietf-mailmaint-oauth-public-07.html#section-6.2) (also -06) | WG Internet-Draft; **member removed in [-08](https://www.ietf.org/archive/id/draft-ietf-mailmaint-oauth-public-08.html#section-6.2)** | `username` | ID (login name, typically the email address) | Unconditional MUST in -06 and -07; absent from -08 | Draft text; no example response in section 6.2 |
| [GNAP, RFC 9635 section 3.4](https://www.rfc-editor.org/rfc/rfc9635.html#section-3.4) | RFC; not OAuth 2.0 | `subject` containing `sub_ids`, `assertions`, `updated_at` | ID | Optional; requested with a `subject` request object, not a scope | RFC text, verbatim example |
| [SMART App Launch 2.2.0](https://hl7.org/fhir/smart-app-launch/scopes-and-launch-context.html#launch-context-arrives-with-your-access_token) | HL7 STU 2.2 | `patient`, `encounter`, `fhirContext`, `need_patient_banner`, `intent`, `smart_style_url`, `tenant` | Context | Depends on `launch`, `launch/patient`, `launch/encounter` scopes and EHR discretion | Spec text, verbatim examples; `tenant` wording via summarising fetch |
| [RFC 9396 (RAR) section 7.1](https://www.rfc-editor.org/rfc/rfc9396.html#section-7.1) | RFC | `authorization_details` | Context (account identifiers such as IBANs) | When the AS enriches the granted details | RFC text |
| [draft-mcguinness-oauth-resource-token-resp-03](https://www.ietf.org/archive/id/draft-mcguinness-oauth-resource-token-resp-03.txt) | Individual draft, expired 24 September 2026 | `resource` | Context (resource identifier) | Not recorded | Draft text |
| [draft-spencer-oauth-claims-00](https://www.ietf.org/archive/id/draft-spencer-oauth-claims-00.txt) | Individual draft, expired May 2020 | `claims` | Neither: space-separated claim *names* granted, not values | Required when it differs from the request | Draft text |
| [UMA 2.0 Grant section 3.3.5](https://docs.kantarainitiative.org/uma/wg/rec-oauth-uma-grant-2.0.html#uma-grant-type) | Kantara Recommendation | `pct`, `upgraded` | Neither: opaque correlation handle | Optional | Summarising fetch |

### IndieAuth: `me` and `profile`

`me` is "the canonical user profile URL for the user this access token corresponds to" and is always returned; `profile` is optional and carries `name`, `photo`, `url` (scope `profile`) and `email` (scope `email`, which must accompany `profile`) ([IndieAuth 5.3.3](https://indieauth.spec.indieweb.org/#access-token-response), [5.3.4](https://indieauth.spec.indieweb.org/#profile-information)). The scope and MAY wording of 5.3.4 was read through a summarising fetch; the property list and the examples are from the raw text. Neither member is in the IANA registry.

```json
{
  "access_token": "XXXXXX",
  "token_type": "Bearer",
  "scope": "profile email create",
  "me": "https://user.example.net/",
  "profile": {
    "name": "Example User",
    "url": "https://user.example.net/",
    "photo": "https://user.example.net/photo.jpg",
    "email": "user@example.net"
  }
}
```

**Trust rules.** On `profile`: "The client MUST NOT treat the information in the profile object as canonical or authoritative, and MUST NOT make any authentication or identification decisions based on this information... any user can create an authorization endpoint that returns any email address in the profile response." On `me`: "the client MUST verify the authorization server is authorized to make claims about the profile URL returned by confirming the returned profile URL declares the same authorization server" ([IndieAuth 5.4](https://indieauth.spec.indieweb.org/#authorization-server-confirmation)).

### AT Protocol OAuth profile: `sub`

"At the end of the auth flow, when the client does an initial token fetch, the Authorization Server must return the account DID in the `sub` field of the JSON response body." The `atproto` scope is required for all sessions, with semantics "similar to the `openid` scope" ([Identity Authentication](https://atproto.com/specs/oauth#identity-authentication), [Authorization Scopes](https://atproto.com/specs/oauth#authorization-scopes)). This is the specification behind Bluesky's OAuth. The page contains no example token response.

**Trust rules.** "At this point, the client still cannot trust that it has actually authenticated the indicated account." Clients must resolve the DID, confirm the declared PDS resolves to the same Authorization Server, and confirm the `issuer`; "it is critical for all clients... to verify that the account identifier (DID) in the `sub` field of the token response is consistent with the Authorization Server hostname/origin (`issuer`)." "Otherwise a malicious server could authenticate arbitrary accounts (DIDs) to the client" ([atproto OAuth](https://atproto.com/specs/oauth)).

### draft-ietf-mailmaint-oauth-public: `username` in -06 and -07 only

Cite this draft by explicit version. **-07 (16 November 2026), section 6.2**, defined: "username — The username of the account that was authenticated and granted this authorization, in the form the user would enter it to log in to this deployment (typically the user's email address). Servers MUST include this property in every token response, and the value MUST be the same in every token response issued under the same authorization" ([-07 section 6.2](https://www.ietf.org/archive/id/draft-ietf-mailmaint-oauth-public-07.html#section-6.2)).

The stated rationale: the user may authenticate as a different user than the `login_hint`, and "Without the username property in the token response, the client instance has no interoperable way to discover this." Clients "MUST use the username value, together with the issuer identifier... as the identity of the account the tokens belong to." On OpenID Connect: "this profile does not require OpenID Connect support, and the sub claim it guarantees is an opaque identifier rather than a username the user would recognize." Section 10.2 asked IANA to change the existing `username` registration to "Parameter usage location: token request, token response" ([-07 section 10.2](https://www.ietf.org/archive/id/draft-ietf-mailmaint-oauth-public-07.html#section-10.2)).

The member first appeared in [-06](https://www.ietf.org/archive/id/draft-ietf-mailmaint-oauth-public-06.txt) (September 2026; the notes disagree on whether it is dated the 16th or the 17th) and is absent from [-05](https://www.ietf.org/archive/id/draft-ietf-mailmaint-oauth-public-05.txt) and earlier. **[-08](https://www.ietf.org/archive/id/draft-ietf-mailmaint-oauth-public-08.txt) (18 November 2026) removed it**: section 6.2 lists only `access_token`, `token_type`, `expires_in`, `scope`, `refresh_token` ([datatracker](https://datatracker.ietf.org/doc/draft-ietf-mailmaint-oauth-public/)).

The mailing-list history below was read through a summarising fetch, so quotations are as relayed. The requirement originated with Matt Diephouse (Apple) on 28 August 2026 ([message](https://mailarchive.ietf.org/arch/msg/mailmaint/dC8qcXb6CEV-dYV_cZ1VKSt4o_w/)). Nick Watson (Google) asked on 17 November 2026 why the draft did not reuse the OIDC `email` scope and an ID Token ([message](https://mailarchive.ietf.org/arch/msg/mailmaint/pMuAB83-vaHOxvHKs4gLRVxRLcw/)). Neil Jenkins replied that requiring OIDC means JWTs, JWKS and full validation "just to learn one string", and that `email` and `preferred_username` have the wrong semantics ([message](https://mailarchive.ietf.org/arch/msg/mailmaint/f-FrKvHNUUZ5vSEU_wK9LL0kNSk/)). He then removed `username` to reach last call ([message](https://mailarchive.ietf.org/arch/msg/mailmaint/GpStUR6Jf4cA80I5SFFgmRWLsm8/)); the replacement is an endpoint, discovered through PACC, that the client GETs with the access token ([message](https://mailarchive.ietf.org/arch/msg/mailmaint/M6WlaLDZQ_91nfMWvg4ZfWpmKjU/)). The PACC draft was not read, so the replacement is known only from that message.

### GNAP (RFC 9635): `subject`

```json
"subject": {
  "sub_ids": [ {
    "format": "opaque",
    "id": "XUT2MFM1XBIKJKSDU8QM"
  } ],
  "assertions": [ {
    "format": "id_token",
    "value": "eyj..."
  } ]
}
```

**Trust rules.** "The AS MUST return the subject field only in cases where the AS is sure that the RO and the end user are the same party." "The client instance MUST interpret all subject information in the context of the AS from which the subject information is received... Otherwise, a rogue AS could exploit this to take over a targeted account asserted by a different AS." An email-format identifier "does not indicate that the AS has validated that the represented email address... is suitable for communication" ([RFC 9635 section 3.4](https://www.rfc-editor.org/rfc/rfc9635.html#section-3.4)).

### SMART App Launch: context members, not the user

```
{
 "access_token": "secret-xyz",
 "patient": "123",
 "fhirContext": [{"reference": "DiagnosticReport/123"}, {"reference": "Organization/789"}],
//...
}
```

SMART deliberately keeps the user out of the plain members: the user is the `fhirUser` claim inside the `id_token`, requested with `openid fhirUser` ([identity data](https://hl7.org/fhir/smart-app-launch/scopes-and-launch-context.html#scopes-for-requesting-identity-data)). `patient` is a context identifier; in patient-facing standalone apps it is in practice the logged-in user's own record, which is the notes' inference rather than spec text. Vendor implementations (Epic, CMS Blue Button, Medplum) are in the context-identifier section below.

## Vendor implementations: 28 OAuth 2.0 token endpoints return a user identifier

| Vendor | Member(s) | Contents | Condition | Evidence |
|---|---|---|---|---|
| [Alipay](https://opendocs.alipay.com/open/02ailc) | `user_id`, `open_id` | 16-digit Alipay user ID starting 2088; app-scoped user ID | Which is returned depends on the app's openid setting; `user_id` is being withdrawn for new merchants | Docs data read from the page's backing endpoint; no example |
| [Bitly](https://dev.bitly.com/docs/getting-started/authentication/) | `login` | Bitly username | Default body is form-encoded; JSON with `Accept: application/json` | Docs |
| [Calendly](https://developer.calendly.com/api-docs/calendly-o-auth/o-auth/post-oauth-refresh-token) | `owner` (plus `organization`, Context) | User URI; organization URI | Both required in the response schema | Docs |
| [Disqus](https://disqus.com/api/docs/auth/) | `user_id`, `username` | Stable ID (string); mutable username | Not stated | Docs; example sits next to an app-registration request, so the plain code-exchange response was not separately verified |
| [Douyin](https://developer.open-douyin.com/docs/resource/zh-CN/dop/develop/openapi/account-permission/get-access-token) | `open_id` | App-scoped user ID | Nested in a `data` envelope | Docs |
| [Dropbox](https://github.com/dropbox/dropbox-sdk-java/blob/main/core/src/main/java/com/dropbox/core/DbxAuthFinish.java) | `account_id`, `uid` (plus `team_id`, Context) | Account ID; legacy user ID | `uid` required by the SDK; at least one of `account_id` or `team_id` | **SDK source only** (two vendor SDKs); no docs example |
| [Figma](https://developers.figma.com/docs/rest-api/oauth-apps/) | `user_id_string`; `user_id` (deprecated) | User ID as string; legacy numeric ID | `user_id` "may" still appear | Docs (placeholder structure) |
| [Fitbit](https://dev.fitbit.com/build/reference/web-api/authorization/oauth2-token/) | `user_id` | Encoded Fitbit user ID (string) | Same response for code exchange and refresh | Docs |
| [Heroku](https://devcenter.heroku.com/articles/oauth) | `user_id` | UUID of the authorizing user | Code exchange and refresh | Docs; member has no prose description |
| [Instagram API with Instagram Login](https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/business-login) | `user_id` | Instagram app-scoped user ID (string) | Wrapped in a `data` array; [not returned on long-lived token refresh](https://developers.facebook.com/docs/instagram-platform/reference/refresh_access_token) | Docs |
| [Komoot](https://static.komoot.de/doc/auth/oauth2.html) | `username` | Komoot username, needed to build API paths | Not stated | Docs |
| [Mercado Libre](https://developers.mercadolibre.com.ar/en_us/authentication-and-authorization) | `user_id` | Numeric user (seller) ID, JSON number | Code exchange and refresh | Docs |
| [Mercado Pago](https://www.mercadopago.com.ar/developers/es/reference/oauth/_oauth_token/post) | `user_id` (plus `public_key`, `live_mode`) | Numeric seller account ID, JSON number | Not stated | OpenAPI data embedded in the reference page; no example |
| [Miro](https://developers.miro.com/reference/exchange-authorization-code-with-access-token) | `user_id` (plus `team_id`, Context) | User ID (string); team ID | Not stated | OpenAPI schema with example values; no literal response |
| [Podio](https://developers.podio.com/authentication/server_side) | `ref` with `type`, `id` | Typed reference; `"type": "user"` and the user ID | Server-side flow | Docs (placeholder structure) |
| [Polar AccessLink](https://www.polar.com/accesslink-api/) | `x_user_id` | "Polar Ecosystem user identifier", JSON number | Not stated | Docs |
| [QQ Connect](https://wiki.connect.qq.com/%E4%BD%BF%E7%94%A8authorization_code%E8%8E%B7%E5%8F%96access_token) | `openid` | App-scoped user ID | **Only when the request sends `need_openid=1`**; body is form-encoded unless `fmt=json` | Docs; no example with `openid` present |
| [RingCentral](https://developers.ringcentral.com/guide/authentication/auth-code-flow) | `owner_id` | "Extension identifier" (the user-level entity) | Not stated | Docs |
| [Salesforce](https://github.com/forcedotcom/SalesforceMobileSDK-Android/blob/dev/libs/SalesforceSDK/src/com/salesforce/androidsdk/auth/OAuth2.java) | `id` (plus `instance_url`, Context) | Identity URL; last path segment is the user ID, second-to-last the org ID | SDK reads both with a throwing accessor | **SDK source only**; Salesforce docs could not be read |
| [Slack](https://docs.slack.dev/reference/methods/oauth.v2.access) | `authed_user`, `bot_user_id` (plus `team`, `enterprise`, Context) | `authed_user.id` is the user ID; the object also carries the user's own `access_token`, `scope`, `token_type` | `authed_user` in all documented examples; `bot_user_id` with bot scopes | Docs |
| [Threads](https://developers.facebook.com/docs/threads/get-started/get-access-tokens-and-permissions) | `user_id` | Numeric user ID, JSON number | Code exchange | Docs |
| [TikTok](https://developers.tiktok.com/doc/oauth-get-a-user-access-token) | `open_id` | "The TikTok user's unique identifier" (app-scoped) | Listed unconditionally | Docs field table; no example (the page's "Response example" shows only an error body) |
| [VK ID](https://id.vk.com/about/business/go/docs/en/vkid/latest/vk-id/connection/api-description) | `user_id` | User identifier | Code exchange (with `id_token`) and refresh (without) | Docs (page served in Russian; placeholder structure) |
| [WeChat](https://developers.weixin.qq.com/doc/oplatform/en/Website_App/WeChat_Login/Authorized_Interface_Calling_UnionID.html) | `openid`, `unionid` | Per-app user ID; ID shared across one developer account's apps | Refresh response omits `unionid`; condition for `unionid` on code exchange not stated | Docs via summarising fetch |
| [Weibo](https://open.weibo.com/wiki/Oauth2/access_token) | `uid` | Numeric UID as a string | Always shown | Docs |
| [Withings](https://developer.withings.com/developer-guide/v3/integration-guide/dropship-cellular/get-access/access-and-refresh-tokens/) | `userid` | Presumed user ID (the page does not define it) | Nested as `body.userid` | Developer-guide code sample via summarising fetch; no example; type not confirmed |
| [Xiaomi](https://dev.mi.com/docs/passport/en/authorization-code/) | `openId` | "user's openId, can be stored... for verifying the user when they sign in next time" | Body prefixed with `&&&START&&&` | Docs |
| [Yahoo](https://developer.yahoo.com/oauth2/guide/flows_authcode/) | `xoauth_yahoo_guid` | Yahoo user GUID | Code exchange and refresh; **deprecated** in favour of `id_token` | Docs |

Related mechanisms that are not OAuth 2.0 JSON members:

| Vendor | Member(s) | Mechanism | Evidence |
|---|---|---|---|
| [X / Twitter OAuth 1.0a](https://docs.x.com/fundamentals/authentication/api-reference) | `user_id`, `screen_name` | Form-encoded OAuth 1.0a token response; [RFC 5849](https://www.rfc-editor.org/rfc/rfc5849.txt) defines no such member | Docs |
| [Flickr OAuth 1.0a](https://www.flickr.com/services/api/auth.oauth.html) | `user_nsid`, `username`, `fullname` | Form-encoded OAuth 1.0a | Docs |
| [Evernote OAuth 1.0a](https://dev.evernote.com/doc/articles/authentication.php) | `edam_userId` (plus `edam_shard`, `edam_noteStoreUrl`) | Form-encoded OAuth 1.0a; "useful for webhook notifications" | Docs |
| [Etsy Open API v3](https://developers.etsy.com/documentation/essentials/authentication) | none | User ID is a numeric prefix inside the `access_token` and `refresh_token` strings | Docs |

### Entry details and examples

**Weibo, `uid`.** The clearest vendor warning found. "授权用户的UID，本字段只是为了方便开发者，减少一次 user/show 接口调用而返回的，第三方应用不能用此字段作为用户登录状态的识别，只有 access_token 才是用户授权的唯一票据。" (The authorizing user's UID; this field is returned only for developer convenience, to save one call to user/show; third-party apps must not use this field to identify the user's login state; only the access_token is the sole credential of user authorization.) ([Weibo](https://open.weibo.com/wiki/Oauth2/access_token))

```json
{
  "access_token": "ACCESS_TOKEN",
  "expires_in": "7200",
  "remind_in": "7200",
  "uid": "1404376560"
}
```

**WeChat, `openid` and `unionid`.** `openid` is the "Unique identifier of the authorized user"; for `unionid`, "For apps under one WeChat Open Platform account, the unionid of the same user is unique." Example as relayed by the fetch tool (indentation not byte-exact) from the [UnionID page](https://developers.weixin.qq.com/doc/oplatform/en/Website_App/WeChat_Login/Authorized_Interface_Calling_UnionID.html); the [WeChat Login overview](https://developers.weixin.qq.com/doc/oplatform/en/Website_App/WeChat_Login/Wechat_Login.html) shows the same six members. One earlier research pass marked WeChat unconfirmed because the page is client-rendered; the follow-up pass confirmed it from the English pages.

```json
{
    "access_token": "ACCESS_TOKEN",
    "expires_in": 7200,
    "refresh_token": "REFRESH_TOKEN",
    "openid": "OPENID",
    "scope": "SCOPE",
    "unionid": "UNIONID"
}
```

**QQ Connect, `openid`.** One research pass found no token response on the page; the other read the parameter table: `need_openid=1` is "表示同时获取openid" (obtain the openid at the same time). Without it, the OpenID comes from a separate `oauth2.0/me` call ([QQ Connect](https://wiki.connect.qq.com/%E4%BD%BF%E7%94%A8authorization_code%E8%8E%B7%E5%8F%96access_token)). This is the only vendor found where the client opts in with a request parameter.

**Alipay, `user_id` and `open_id`.** "新商户建议使用open_id替代该字段。对于新商户，user_id字段未来计划逐步回收，存量商户可继续使用。" (New merchants are advised to use open_id instead; for new merchants user_id is planned to be gradually withdrawn.) ([Alipay](https://opendocs.alipay.com/open/02ailc))

**Douyin, `open_id`.** The first two lines are reconstructed from the documented structure; the rest is from the page ([Douyin](https://developer.open-douyin.com/docs/resource/zh-CN/dop/develop/openapi/account-permission/get-access-token)).

```json
{
    "data": {
        "access_token": "act.f7094fbffab2ecbfc45e9af9c32bc241oYdckvBKe82BPx8T******",
        "captcha": "",
        "desc_url": "",
        "description": "",
        "error_code": 0,
        "expires_in": 1296000,
        "log_id": "20230525105733ED3ED7AC56A******",
        "open_id": "b9b71865-7fea-44cc-******",
        "refresh_expires_in": 2592000,
        "refresh_token": "rft.713900b74edde9f30ec4e246b706da30t******",
        "scope": "user_info"
    },
    "message": "success"
}
```

**Xiaomi, `openId`.** The trailing spaces inside two key names are in the original ([Xiaomi](https://dev.mi.com/docs/passport/en/authorization-code/)).

```
&&&START&&& {
  "access_token": "access token value",
  "expires_in": 360000,
  "refresh_token": "refresh token value",
  "scope": "scope value",
  "token_type ": "mac",
  "mac_key ": "mac key value",
  "mac_algorithm": " HmacSha1",
  "openId":"2.0XXXXXXXXX"
}
```

**VK ID, `user_id`.** A hybrid: it returns both an `id_token` and a plain `user_id`. The two research passes cite the [/en/](https://id.vk.com/about/business/go/docs/en/vkid/latest/vk-id/connection/api-description) and [/ru/](https://id.vk.com/about/business/go/docs/ru/vkid/latest/vk-id/connection/api-description) URLs of the same page; both were served in Russian. The legacy `oauth.vk.com` flow and its `email` member are unconfirmed (see the last section).

```
{
  "refresh_token": <refresh token>,
  "access_token": <access token>,
  "id_token": <id token>,
  "token_type": <тип токена — по умолчанию Bearer>,
  "expires_in": <срок действия токена>,
  "user_id": <идентификатор пользователя>,
  "state": <произвольная строка состояния>,
  "scope": <список прав доступа>
}
```

**Yahoo, `xoauth_yahoo_guid`.** "This claim is deprecated. If you need the user's GUID value, please use the OpenID Connect flows. The GUID will be provided in the id_token." ([Yahoo](https://developer.yahoo.com/oauth2/guide/flows_authcode/))

```json
{
  "access_token" : "Jzxbkqqcvjqik2IMxGFEE1cuaos--",
  "token_type" : "bearer",
  "expires_in" : 3600,
  "refresh_token" : "AOiRUlJn_qOmByVGTmUpwcMKW3XDcipToOoHx2wRoyLgJC_RFlA-",
  "xoauth_yahoo_guid" : "JT4FACLQZI2OCE"
}
```

**Figma, `user_id_string`.** "Note: This response may contain a numeric user_id property. This property is deprecated. Please use user_id_string instead. Figma user IDs appear to be numeric, but many cannot be represented in common number formats such as JavaScript numbers and Go's float64." ([Figma](https://developers.figma.com/docs/rest-api/oauth-apps/))

```
{
  "user_id_string": <USER ID for the user who authorized the app>,
  "access_token": <TOKEN>,
  "token_type": "bearer",
  "expires_in": <EXPIRATION (in seconds)>,
  "refresh_token": <REFRESH TOKEN>
}
```

**Heroku, `user_id`.** The `access_token` value is shortened here; the docs show a placeholder token ([Heroku](https://devcenter.heroku.com/articles/oauth)).

```json
{
  "access_token":"HRKU-...",
  "expires_in":28799,
  "refresh_token":"01234567-89ab-cdef-0123-456789abcdef",
  "token_type":"Bearer",
  "user_id":"01234567-89ab-cdef-0123-456789abcdef",
  "session_nonce":"2bf3ec81701ec291"
}
```

**Fitbit, `user_id`** ([Fitbit](https://dev.fitbit.com/build/reference/web-api/authorization/oauth2-token/)).

```json
{
  "access_token": "eyJhbGciOiJIUzI1...",
  "expires_in": 28800,
  "refresh_token": "c643a63c072f0f05478e9d18b991db80ef6061e...",
  "token_type": "Bearer",
  "user_id": "GGNJL9"
}
```

**Threads, `user_id`** as a JSON number ([Threads](https://developers.facebook.com/docs/threads/get-started/get-access-tokens-and-permissions)).

```json
{
  "access_token": "THQVJ...",
  "token_type": "bearer",
  "user_id": 17841405793187218
}
```

**Instagram, `user_id`** as a string inside a `data` array ([Instagram](https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/business-login)).

```
{
  "data": [
    {
      "access_token": "EAACEdEose0...",
      "user_id": "1020...",                  // Your app user's Instagram-scoped user ID
      "permissions": "instagram_business_basic,instagram_business_manage_messages,instagram_business_manage_comments,instagram_business_content_publish"
    }
  ]
}
```

**Mercado Libre, `user_id`** as a JSON number; the ID is also visible as the suffix of both tokens ([Mercado Libre](https://developers.mercadolibre.com.ar/en_us/authentication-and-authorization)). One research pass recorded that refresh behaviour was not stated; two others found `user_id` in both the code-exchange and refresh examples.

```json
{
  "access_token": "APP_USR-123456-090515-8cc4448aac10d5105474e1351-1234567",
  "token_type": "bearer",
  "expires_in": 10800,
  "scope": "offline_access read write",
  "user_id": 1234567,
  "refresh_token": "TG-5b9032b4e23464aed1f959f-1234567"
}
```

**Polar AccessLink, `x_user_id`** ([Polar](https://www.polar.com/accesslink-api/)).

```json
{
  "access_token" : "2YotnFZFEjr1zCsicMWpAA",
  "token_type" : "bearer",
  "expires_in" : 31535999,
  "x_user_id" : 10579
}
```

**Komoot, `username`.** "username of current user can be taken from the token response" for building `/users/USERNAME/tours` requests ([Komoot](https://static.komoot.de/doc/auth/oauth2.html)).

```json
{
  "access_token": "the_access_token",
  "token_type": "bearer",
  "refresh_token": "the_refresh_token",
  "expires_in": 59,
  "scope": "the space separated list of scopes",
  "username": "username_of_the_user",
  "jti": "a338c2f9-4bb0-4912-8a3e-faff4309e28b"
}
```

**Disqus, `username` and `user_id`.** The missing comma after `"batman"` is in the original ([Disqus](https://disqus.com/api/docs/auth/)).

```
{
  "access_token": "c2d06abacfbb40179e47f62f06546ea9",
  "refresh_token": "9182211bf2f746a4b5c5b1e3766443d6",
  "expires_in": 2592000,
  "username": "batman"
  "user_id": "947103743",
  "api_key": "9BGzwXpWUUcJIqbw1jm6NmgylehvSvIWfCqCM63mpkjrAudzhhq7uYGk8ttUxScN",
  "api_secret": "hzUC9yEGgLm9LfbaUdbtzcDhrcFuRPKfyXQC9DwzUWBYFWgD05lASvTB9xgs6Ti3"
}
```

**Bitly, `login`.** Default response: `access_token=TOKEN&login=BITLY_LOGIN` ([Bitly](https://dev.bitly.com/docs/getting-started/authentication/)).

**RingCentral, `owner_id`** ([RingCentral](https://developers.ringcentral.com/guide/authentication/auth-code-flow)).

```json
{
   "access_token" : "U1BCMDFUMDRKV1MwMXxzLFSvXdw5PHMsVLEn_MrtcyxUsw",
   "token_type" : "bearer",
   "expires_in" : 7199,
   "refresh_token" : "U1BCMDFUMDRKV1MwMXxzLFL4ec6A0XMsUv9wLriecyxS_w",
   "refresh_token_expires_in" : 604799,
   "scope" : "AccountInfo CallLog ExtensionInfo Messages SMS",
   "owner_id" : "256440016"
}
```

**Calendly, `owner` and `organization`.** `owner` is "A link to the resource that owns the token (currently, this is always a user)". The `access_token` value is truncated here; other lines are as on the page ([Calendly](https://developer.calendly.com/api-docs/calendly-o-auth/o-auth/post-oauth-refresh-token), [usage guide](https://developer.calendly.com/docs/api-guides/how-to-find-the-organization-or-user-uri)).

```json
{
  "token_type": "Bearer",
  "access_token": "eyJhbGciOiJIUzI1NiJ9...",
  "refresh_token": "b77a76ffce83d3bc20531ddfa76704e584f0ee963f6041b8bfc70c91373267d5",
  "created_at": 1548689183,
  "expires_in": 7200,
  "owner": "https://api.calendly.com/users/EBHAAFHDCAEQTSEZ",
  "organization": "https://api.calendly.com/organizations/EBHAAFHDCAEQTSEZ",
  "scope": "default"
}
```

**Podio, `ref`** ([Podio](https://developers.podio.com/authentication/server_side)).

```
{
  "access_token": ACCESS_TOKEN,
  "token_type": "bearer",
  "expires_in": EXPIRES_IN,
  "refresh_token": REFRESH_TOKEN,
  "scope": GRANTED_SCOPE_STRING,
  "ref":
  {
    "type": "user",
    "id": USER_ID
  }
}
```

**Slack, `authed_user`.** "Successful Sign in with Slack response" from the docs; a second documented example adds a top-level bot `access_token`, `bot_user_id`, and `name` inside `team` and `enterprise` ([Slack](https://docs.slack.dev/reference/methods/oauth.v2.access)). Note that the user's own access token is nested inside the user object.

```json
{
  "ok": true,
  "app_id": "A0118NQPZZC",
  "authed_user": {
    "id": "U065VRX1T0",
    "scope": "identity.basic,identity.email,identity.avatar,identity.team",
    "access_token": "xoxp-yoda-yoda-yoda",
    "token_type": "user"
  },
  "team": {
    "id": "T024BE7LD"
  },
  "enterprise": null,
  "is_enterprise_install": false
}
```

**Dropbox, `account_id`, `uid`, `team_id`.** Two earlier passes could not confirm Dropbox because its docs are client-rendered; the follow-up confirmed the members from Dropbox's own SDKs, which is the only basis. The Java SDK throws on a missing `uid` and on "missing field \"account_id\" and missing field \"team_id\"" when both are absent; its Javadoc calls `accountId` an "Obfusticated user or team id" ([Java SDK](https://github.com/dropbox/dropbox-sdk-java/blob/main/core/src/main/java/com/dropbox/core/DbxAuthFinish.java)). The Python SDK labels `user_id` "Deprecated (use account_id instead)", as relayed by the fetch tool ([Python SDK](https://github.com/dropbox/dropbox-sdk-python/blob/main/dropbox/oauth.py)).

**Salesforce, `id` and `instance_url`.** An earlier pass marked this unconfirmed; the follow-up confirmed it from the Salesforce Mobile SDK for Android, which reads both members with a throwing accessor and splits `id` into user ID and org ID ([OAuth2.java](https://github.com/forcedotcom/SalesforceMobileSDK-Android/blob/dev/libs/SalesforceSDK/src/com/salesforce/androidsdk/auth/OAuth2.java)). "Always returned" is inferred from that accessor, not from a documentation statement, and presence across grant types was not checked.

**TikTok, `open_id`.** The token-management page tells developers to "Store the following values on your server for each user:" `access_token`, `refresh_token`, `open_id`, `scope` ([TikTok](https://developers.tiktok.com/doc/oauth-user-access-token-management)).

**X / Twitter OAuth 1.0a** ([X](https://docs.x.com/fundamentals/authentication/api-reference)).

```
oauth_token=6253282-eWudHldSbIaelX7swmsiHImEL4KinwaGloHANdrY&oauth_token_secret=2EEfA6BG5ly3sR3XjE0IBSnlQu4ZrUzPiYTmrkVU&user_id=6253282&screen_name=xapi
```

**Flickr OAuth 1.0a** ([Flickr](https://www.flickr.com/services/api/auth.oauth.html)).

```
fullname=Jamal%20Fanaian
&oauth_token=72157626318069415-087bfc7b5816092c
&oauth_token_secret=a202d1f853ec69de
&user_nsid=21207597%40N07
&username=jamalfanaian
```

**Etsy.** "access_token is the OAuth grant token with a user id numeric prefix (12345678 in the example above), which is the internal user_id of the Etsy.com user who grants the application access." ([Etsy](https://developers.etsy.com/documentation/essentials/authentication))

## Vendor implementations: ten return a profile object beside the access token

| Vendor | Member(s) | Contents | Condition | Evidence |
|---|---|---|---|---|
| [Asana](https://developers.asana.com/docs/oauth) | `data` | `id`, `gid`, `name`, `email` of the logged-in user | Shown for code exchange; refresh not stated | Docs |
| [BigCommerce](https://docs.bigcommerce.com/docs/integrations/apps/guide/auth) | `user`, `owner` (plus `context`, `account_uuid`, Context) | `id`, `username`, `email` of the authorizing user and of the store owner | "A modified version of the OAuth2 authorization code grant" | Docs |
| [DigiLocker](https://img1.digitallocker.gov.in/assets/img/Digital%20Locker%20Authorized%20Partner%20API%20Specification%20v1.11.pdf) (India) | `digilockerid`, `name`, `dob`, `gender`, `eaadhaar`, `new_account`, `reference_key` | 36-character user ID plus demographics, as flat top-level members | No scope condition documented | Spec v1.11 PDF read from an [archived copy](https://web.archive.org/web/2023/https://img1.digitallocker.gov.in/assets/img/Digital%20Locker%20Authorized%20Partner%20API%20Specification%20v1.11.pdf); the direct URL timed out |
| [DigitalOcean](https://docs.digitalocean.com/reference/api/oauth/) | `info` | `name`, `email`, `uuid` (user); `team_uuid`, `team_name` | Not stated | Docs |
| [Feishu / Lark v1](https://open.feishu.cn/document/server-docs/authentication-management/access-token/create-2) | `name`, `en_name`, `avatar_url`, `open_id`, `union_id`, `user_id`, `email`, `mobile`, `tenant_key` and more, inside `data` | Full profile plus three user IDs and a tenant key | `email` and `mobile` gated by granted permissions; **endpoint deprecated**, and [v2](https://open.feishu.cn/document/authentication-management/access-token/get-user-access-token) dropped all of it | Docs; no example |
| [Notion](https://developers.notion.com/reference/create-a-token) | `owner` (plus `workspace_id`, `workspace_name`, `workspace_icon`, `bot_id`, `duplicated_template_id`) | User object for the authorizing user | Schema marks all as required; `owner` is `anyOf` a user object or `{"type":"workspace","workspace":true}` | OpenAPI schema and [field table](https://developers.notion.com/docs/authorization); no example |
| [Shopify](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/online-access-tokens) | `associated_user`, `associated_user_scope` | `id`, `account_owner`, `collaborator`, `email`, `email_verified` | **Online (per-user) tokens only**; offline tokens carry no user data | Docs; no online-token example found |
| [Strava](https://developers.strava.com/docs/authentication/) | `athlete` | "A summary of athlete information" | **Code exchange only**; not in the refresh response | Docs |
| [Vimeo](https://github.com/vimeo/openapi/blob/master/api.yaml) | `user` (plus `app`) | Full user object: "The user associated with the token." | Not in the schema's `required` list; the same schema serves client credentials | Vendor OpenAPI spec (last commit 2023-12-13); no example |
| [Yammer / Viva Engage](https://learn.microsoft.com/en-us/rest/api/yammer/oauth-2) | `user`, `network`; `access_token` is itself an object with `token`, `user_id`, `network_id` | Full profile, network object | **Retired**: legacy OAuth 2 restricted from 1 July 2025 | Docs |

### Entry details and examples

**Asana, `data`.** "An object encoding a few key fields about the logged-in user. Currently, this is the user's `id`, `gid`, `name`, and `email`" ([Asana](https://developers.asana.com/docs/oauth)).

```json
{
  "access_token": "f6ds7fdsa69ags7ag9sd5a",
  "expires_in": 3600,
  "token_type": "bearer",
  "data": {
    "id": 4673218951,
    "gid": "4673218951",
    "name": "Greg Sanchez",
    "email": "gsanchez@example.com"
  },
  "refresh_token": "hjkl325hjkl4325hj4kl32fjds"
}
```

**DigitalOcean, `info`.** The email is obfuscated in the raw HTML by Cloudflare; the value shown is as rendered ([DigitalOcean](https://docs.digitalocean.com/reference/api/oauth/)).

```json
{
    "access_token": "doo_v1_EXAMPLE4efe8999d0892b6069bc754a78c656f8e843361e1e4d1cd04ac85c381",
    "token_type": "bearer",
    "expires_in": 2592000,
    "refresh_token": "dor_v1_EXAMPLE3104521c47be0b580e9296453ef4y319b02b5513469f0ec72d99af2e2",
    "scope": "read",
    "info": {
        "name": "Sammy the Shark",
        "email":"sammy@digitalocean.com",
        "uuid":"EXAMPLE0-a636-11ec-9e9d-3381ceabe039",
        "team_uuid": "EXAMPLE1-a636-11ec-a6ac-1323bf96ef4d",
        "team_name": "My Team"
    }
}
```

**BigCommerce, `user` and `owner`.** `user.id`: "BigCommerce's unique identifier for the authorized user. Save this value to identify the user in future requests." ([BigCommerce](https://docs.bigcommerce.com/docs/integrations/apps/guide/auth))

```json
{
  "access_token": "xxxxalphanumstringxxxx",
  "scope": "store_v2_orders store_channel_listings_read_only",
  "user": {
    "id": 24654,
    "username": "merchant@example.com",
    "email": "merchant@example.com"
  },
  "owner": {
    "id": 12345,
    "username": "owner@example.com",
    "email": "owner@example.com"
  },
  "context": "stores/g5cd38",
  "account_uuid": "12345678-90ab-cdef-1234-567890abcdef"
}
```

**Shopify, `associated_user`.** The only vendor warning about email trust: "`email_verified`: Whether the user's email is verified. The `email` field is always present, so trust it only when `email_verified` is `true`." Also "`id`: Uniquely identifies the user. Use it to key per-user data." ([Shopify online access tokens](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/online-access-tokens); the [authorization code grant page](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/authorization-code-grant) shows only the offline response.)

**Strava, `athlete`.** The docs use a placeholder for the object ([Strava](https://developers.strava.com/docs/authentication/)).

```
{
  "token_type": "Bearer",
  "expires_at": 1568775134,
  "expires_in": 21600,
  "refresh_token": "e5n567567...",
  "access_token": "a4b945687g...",
  "athlete": {
    #{summary athlete representation}
  },
  "scope": "activity:read activity:write"
}
```

**Notion, `owner`.** The two research passes differ on evidence quality: one quotes the field table ("A user object is returned, representing the user who authorized the connection."), the other records the descriptions as a fetch-tool summary. Both agree the OpenAPI schema is the firm source and that no literal example exists. Notion's advice: "Store all of the information that your connection receives with the `access_token` and `refresh_token`... It's really hard (or impossible) to send users to repeat the authorization flow to generate the information again." ([Notion authorization](https://developers.notion.com/docs/authorization))

**DigiLocker.** The clearest analogue of plain profile claims in a token response, though flat rather than nested. Revision history: v1.6 (14/07/2019) "Added DigiLocker Id and user details in Get Access Token response". Sample from spec v1.11, extracted from the archived PDF with whitespace normalised; the valueless `"scope":,` is in the original.

```
{
    "access_token": "bc125c212a4b03a9a188a858be5a163f379e878a",
    "expires_in": 3600,
    "token_type": "Bearer",
    "scope":,
    "refresh_token": "a47ab18c593703e4f83a274694db7422a8cfcb8f",
    "digilockerid": "123e4567-e89b-12d3-a456-426655440000",
    "name": "Sunil Kumar",
    "dob": "31121970",
    "gender": "M",
    "eaadhaar": "Y",
    "new_account": "Y",
    "reference_key": "2a33349e7e606a8ad2e30e3c84521f9377450cf09083e162e0a9b1480ce0f972"
}
```

**Vimeo, `user`.** An earlier pass could not read Vimeo's client-rendered docs; the follow-up confirmed the member from Vimeo's own OpenAPI file, where the 200 response of `POST /oauth/access_token` uses the `auth` schema with required members `access_token`, `app`, `scope`, `token_type` and an optional `user` ([vimeo/openapi](https://github.com/vimeo/openapi/blob/master/api.yaml)). The repository may lag the live API.

**Yammer.** "Yammer will return an access token object as part of the response, which includes user profile information. From this object, parse out and store the "token" property." ([Yammer](https://learn.microsoft.com/en-us/rest/api/yammer/oauth-2))

## Tenant, merchant, patient and other context identifiers

These members identify what the token acts on, not who authorized it. Several vendors in the earlier tables return one alongside the user: Slack `team` and `enterprise`, Notion `workspace_id`, Miro `team_id`, Calendly `organization`, DigitalOcean `info.team_uuid`, BigCommerce `context` and `account_uuid`, Dropbox `team_id`, Salesforce `instance_url`, Feishu v1 `tenant_key`.

| Vendor | Member(s) | Contents | Condition | Evidence |
|---|---|---|---|---|
| [CMS Blue Button 2.0](https://bluebutton.cms.gov/api-documentation/authorization/) | `patient`, `access_grant_expiration` | Beneficiary's FHIR patient ID (effectively the authorizing user); grant expiry | Not stated | Docs |
| [Discord](https://docs.discord.com/developers/topics/oauth2) | `webhook`; `guild` | Webhook object (channel, guild); guild object | `webhook.incoming` scope; advanced bot authorization. The standard user flow returns no identifier | Docs |
| [Epic](https://fhir.epic.com/Documentation?docId=oauth2) | `patient`, `__epic.dstu2.patient`, `encounter`, `appointment`, `location`, `loginDepartment` | FHIR IDs for the launch context | Depends on launch type and scopes; `id_token` only with `openid` | Docs |
| [Medplum](https://github.com/medplum/medplum/blob/main/packages/server/src/oauth/token.ts) | `project`, `profile`, `patient`, `encounter`, `fhirContext` | `project` is a FHIR Reference to the tenant; `profile` is a FHIR Reference to the user's own profile resource (a user identifier) | SMART launch context; FHIRcast members with FHIRcast scopes | **Server source only**; the [public docs](https://www.medplum.com/docs/api/oauth/token) show only standard members |
| [Plaid](https://plaid.com/docs/api/items/#itempublic_tokenexchange) | `item_id` | Identifier of one user's connection to one institution | Not an OAuth token endpoint; analogous token exchange | Docs |
| [Square](https://developer.squareup.com/reference/square/oauth-api/obtain-token) | `merchant_id` | ID of the authorizing merchant | Not stated; `id_token` deprecated | Docs example; field semantics via summarising fetch |
| [Stripe Connect](https://docs.stripe.com/connect/oauth-reference) | `stripe_user_id` | Connected *account* ID despite the name | `access_token`, `refresh_token`, `stripe_publishable_key` are all marked deprecated, so the ID is the main output | Docs field table; no example |
| [Vercel integrations](https://vercel.com/docs/rest-api/vercel-api-integrations) | `team_id`, `installation_id` | Team the integration was installed on (null if personal); installation ID | Long-lived tokens from this exchange are deprecated | Docs prose only; the example is collapsed on the page, so **`user_id` is unconfirmed** |
| [WordPress.com](https://developer.wordpress.com/docs/oauth2/) | `blog_id`, `blog_url` | Site the token is scoped to | Code and password grants; the user ID comes from a separate `token-info` call | Docs |

Deployment-routing members reveal the tenant or region without identifying anyone: Zoho `api_domain` ([docs](https://www.zoho.com/accounts/protocol/oauth/web-apps/access-token.html)), Wrike `host` ([docs](https://developers.wrike.com/oauth-20-authorization/)), Pipedrive `api_domain`, which includes the `company_domain` ([docs](https://pipedrive.readme.io/docs/marketplace-oauth-authorization)), and Zoom `api_url` ([docs](https://developers.zoom.us/docs/integrations/oauth/)).

**Intuit QuickBooks `realmId` is not a token response member.** It arrives as a query parameter on the redirect URI; Intuit's Node client copies it into its local token object, which is why it looks like one. Confirmed from Intuit's SDK source and README only; Intuit's documentation pages could not be read ([OAuthClient.js](https://github.com/intuit/oauth-jsclient/blob/master/src/OAuthClient.js), [README](https://github.com/intuit/oauth-jsclient/blob/master/README.md), [Python client](https://github.com/intuit/oauth-pythonclient/blob/master/intuitlib/client.py)).

### Entry details and examples

**Square, `merchant_id`** ([Square](https://developer.squareup.com/reference/square/oauth-api/obtain-token)).

```json
{
  "access_token": "EAAl3ikZIe18J-2-cHlV2bL4-EaZHGoJUhtEBT7QA6-7AgwIHw8Xe1IoUvGsNxA",
  "token_type": "bearer",
  "expires_at": "2025-04-03T18:31:06Z",
  "merchant_id": "MLQW2MYBY81PZ",
  "refresh_token": "EQAAl0OcByu3IYJYScGGg-8E5YNf0r0b6jCTCMy5nOcRZ4ok0wbWAL8vY3tZWNcc",
  "short_lived": false
}
```

**WordPress.com, `blog_id` and `blog_url`** ([WordPress.com](https://developer.wordpress.com/docs/oauth2/)).

```json
{
    "access_token": "YOUR_API_TOKEN",
    "blog_id": "blog_id_number",
    "blog_url": "https://yourblog.wordpress.com",
    "token_type": "bearer"
}
```

**CMS Blue Button 2.0, `patient`.** The opening brace and the `"access_token"` key were cut off in extraction and are reconstructed; values are from the page ([Blue Button](https://bluebutton.cms.gov/api-documentation/authorization/)).

```json
{
  "access_token": "VD1VaT4IfjXAMlZTS9E4RVXZlkhYG7",
  "expires_in": 36000,
  "token_type": "Bearer",
  "scope": "profile patient/Patient.rs patient/Coverage.rs patient/ExplanationOfBenefit.rs",
  "refresh_token": "7x0VkRQlRU4fRNCQL2vh239nIyucgw",
  "patient": "-20140000000001",
  "access_grant_expiration": "2025-09-05 19:17:53Z"
}
```

**Epic.** Token values are shortened here; member names and the trailing comma after `state` are as on the page ([Epic](https://fhir.epic.com/Documentation?docId=oauth2)).

```
{
"access_token": "Nxfve4q3H9TKs5F5vf6kRYAZqzK7j9LHvrg1Bw7fU_07_FdV9aRz...",
"token_type": "bearer",
"expires_in": 3240,
"scope": "openid Patient.read Patient.search ",
"id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IktleUlEIiwidHlwIjoiSldUIn0...",
"__epic.dstu2.patient": "T1wI5bk8n1YVgvWk9D05BmRV0Pi3ECImNSK8DKyKltsMB",
"patient": "T1wI5bk8n1YVgvWk9D05BmRV0Pi3ECImNSK8DKyKltsMB",
"appointment": "eVa-Ad1SCIenVq8CYQVxfKwGtP-DfG3nIy9-5oPwTg2g3",
"encounter": "eySnzekbpf6uGZz87ndPuRQ3",
"location": "e4W4rmGe9QzuGm2Dy4NBqVc0KDe6yGld6HW95UuN-Qd03",
"loginDepartment": "e4W4rmGe9QzuGm2Dy4NBqVc0KDe6yGld6HW95UuN-Qd03",
"state": "abc123",
}
```

**Plaid, `item_id`** ([Plaid](https://plaid.com/docs/api/items/#itempublic_tokenexchange)).

```json
{
  "access_token": "access-sandbox-de3ce8ef-33f8-452c-a685-8671031fc0f6",
  "item_id": "M5eVJqLnv3tbzdngLDp9FL5OlDNxlNhlE55op",
  "request_id": "Aim3b"
}
```

**Discord, `webhook`.** With the `webhook.incoming` scope "you will receive a slightly modified token response" containing a `webhook` object with `application_id`, `name`, `url`, `channel_id`, `token`, `type`, `avatar`, `guild_id`, `id` ([Discord](https://docs.discord.com/developers/topics/oauth2)).

**Stripe Connect, `stripe_user_id`.** "The unique ID of the account you've been granted access to, as a string." The token endpoint section is titled "Complete the connection and get the account ID" ([Stripe](https://docs.stripe.com/connect/oauth-reference)).

## Authorization server products: one built-in member, six extension hooks

| Product | Support | Member or hook | Evidence |
|---|---|---|---|
| [FusionAuth](https://fusionauth.io/docs/apis/oauth/token) | **Built-in** | `userId`: "The unique Id of the user that has been authenticated." Returned on authorization code and refresh grants | Docs |
| [Keycloak](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/mappers/OIDCAccessTokenResponseMapper.java) | Hook, configuration-level | `OIDCAccessTokenResponseMapper.transformAccessTokenResponse(...)`; mapper switch `access.tokenResponse.claim`; server-assigned members are protected ([helper](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/mappers/OIDCAttributeMapperHelper.java)) | **Source only**; no documentation page located |
| [PingAM (ForgeRock AM)](https://docs.pingidentity.com/pingam/8/am-oauth2/modifying-access-tokens-scripts.html) | Hook, script | `accessToken.addExtraData("hello", "world")` adds a member to the `/oauth2/access_token` response | Docs |
| [Duende IdentityServer](https://docs.duendesoftware.com/identityserver/reference/validators/custom-token-request-validator/) | Hook, code | `ICustomTokenRequestValidator`: "The `CustomResponse` dictionary allows emitting additional response fields." | Docs |
| [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper/wiki/Customizing-Token-Response) | Hook, code | Override `Doorkeeper::OAuth::TokenResponse#body`; the wiki's worked example adds `username` and `userid` | Project wiki |
| [league/oauth2-server](https://github.com/thephpleague/oauth2-server/blob/master/src/ResponseTypes/BearerTokenResponse.php) (under Laravel Passport) | Hook, code | `BearerTokenResponse::getExtraParams()` | **Source only**; Laravel Passport's own docs not checked |
| [Spring Authorization Server](https://docs.spring.io/spring-authorization-server/reference/protocol-endpoints.html) | Hook, code | `accessTokenResponseHandler()`; response model carries `additionalParameters` ([source](https://github.com/spring-projects/spring-security/blob/main/oauth2/oauth2-core/src/main/java/org/springframework/security/oauth2/core/endpoint/OAuth2AccessTokenResponse.java)) | Docs and source |
| [Ory Hydra](https://www.ory.com/docs/hydra/guides/claims-at-refresh) | No hook found | Webhook customises token claims; nothing on the page about top-level response members | Docs |
| [Auth0](https://auth0.com/docs/customize/actions/explore-triggers/signup-and-login-triggers/login-trigger/post-login-api-object) | No documented way (Actions) | Post-login Action API changes claims and scopes inside tokens only | Docs via summarising fetch; legacy Rules and other triggers not checked |
| [Zitadel](https://zitadel.com/docs/apis/openidoauth/endpoints) | None | Standard members and `id_token` | Docs |
| [Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/token-endpoint.html), [Okta](https://developer.okta.com/docs/guides/implement-grant-type/authcode/main/), [Microsoft Entra](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow) | None | Standard members and `id_token` | Docs |

**FusionAuth, `userId`.** "This user Id may be used to retrieve the entire user object using the /api/user API." JWT values are shortened here ([FusionAuth](https://fusionauth.io/docs/apis/oauth/token)).

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 3600,
  "id_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "ze9fi6Y9sMSf3yWp3aaO2w7AMav2MFdiMIi2GObrAi-i3248oo0jTQ",
  "refresh_token_id": "4c8927f1-95cd-4bf8-9534-066389ffff5e",
  "scope": "openid offline_access",
  "token_type": "Bearer",
  "userId": "3b6d2f70-4821-4694-ac89-60333c9c4165"
}
```

**PingAM** ([PingAM](https://docs.pingidentity.com/pingam/8/am-oauth2/modifying-access-tokens-scripts.html)).

```json
{
  "access_token":"sbQZuveFumUDV5R1vVBl6QAGNB8",
  "hello":"world",
  "scope":"write",
  "token_type":"Bearer",
  "expires_in":3599
}
```

Keycloak's hook is the only one that works by configuration (a switch on a protocol mapper), so a Keycloak deployment could emit a plain `sub` member without custom code. That reading comes from source, not documentation.

## Observations for the draft

### Member names: `user_id` leads, and only AT Protocol uses `sub`

| Name family | Where it appears |
|---|---|
| `user_id` | Heroku, Mercado Libre, Mercado Pago, Miro, Instagram, Threads, Fitbit, VK ID, Alipay, Disqus; also X OAuth 1.0a, Figma (deprecated), Feishu v1 |
| `open_id` / `openid` / `openId` | TikTok, Douyin, Alipay, Feishu v1 (`open_id`); WeChat, QQ Connect (`openid`); Xiaomi (`openId`) |
| `uid` | Weibo, Dropbox |
| `username` / `login` | Komoot, Disqus, Bitly (`login`), mailmaint -06/-07, Doorkeeper's wiki example |
| `userId` / `userid` | FusionAuth; Withings, Doorkeeper's wiki example |
| `owner` / `owner_id` | Calendly, Notion; RingCentral |
| `sub` | AT Protocol only |
| One-off names | `me` (IndieAuth), `account_id` (Dropbox), `id` (Salesforce), `user_id_string` (Figma), `x_user_id` (Polar), `xoauth_yahoo_guid` (Yahoo), `digilockerid` (DigiLocker), `ref` (Podio), `authed_user` (Slack) |

Profile objects are equally unharmonised: `profile` (IndieAuth), `data` (Asana), `info` (DigitalOcean), `user` (BigCommerce, Vimeo, Yammer), `associated_user` (Shopify), `owner` (Notion), `athlete` (Strava). **No confirmed vendor or specification uses `userinfo`** as a token response member.

### AT Protocol's `sub` is compatible in shape but differs in scope and trigger

Both are a top-level string identifying the authorizing user. AT Protocol's value is a globally resolvable DID rather than an AS-local identifier, and it is triggered by the `atproto` scope ([atproto OAuth](https://atproto.com/specs/oauth#authorization-scopes)). A server implementing both would return one `sub` with consistent meaning, so the collision looks benign, but any rule in the draft about when `sub` must or must not appear should be checked against atproto servers that return it without the draft's trigger. This compatibility assessment is the research notes' inference, not a statement from either specification.

### Strings versus numbers

| Type | Examples |
|---|---|
| JSON string | Fitbit `user_id`, Instagram `user_id`, Heroku `user_id`, Weibo `uid`, Disqus `user_id`, RingCentral `owner_id`, Miro `user_id`, Figma `user_id_string`, FusionAuth `userId`, atproto `sub` |
| JSON number | Threads `user_id`, Mercado Libre `user_id`, Mercado Pago `user_id`, Polar `x_user_id`, Asana `data.id`, BigCommerce `user.id`, Yammer `user.id` |
| Both | Asana returns numeric `id` and string `gid`; Figma returns `user_id_string` and may still return numeric `user_id` |
| URL | IndieAuth `me`, Calendly `owner`, Salesforce `id` |

Meta's two products disagree with each other: Threads returns a number and Instagram a string. Figma's deprecation note is direct evidence for a string-typed identifier ([Figma](https://developers.figma.com/docs/rest-api/oauth-apps/)).

### Security guidance worth quoting

| Source | Guidance |
|---|---|
| [Weibo](https://open.weibo.com/wiki/Oauth2/access_token) | `uid` is a convenience to save an API call; apps must not use it to identify the user's login state, and should validate the token through `oauth2/get_token_info` |
| [AT Protocol](https://atproto.com/specs/oauth#identity-authentication) | After receiving `sub`, "the client still cannot trust that it has actually authenticated the indicated account" until it verifies the DID against the issuer |
| [IndieAuth](https://indieauth.spec.indieweb.org/#profile-information) | Identifier may be keyed on after an authority check; `profile` must never drive "authentication or identification decisions" |
| [GNAP](https://www.rfc-editor.org/rfc/rfc9635.html#section-3.4) | Interpret subject information only in the context of the issuing AS; an email-format identifier is not a verified contact address |
| [mailmaint -07](https://www.ietf.org/archive/id/draft-ietf-mailmaint-oauth-public-07.html#section-6.2) | Bind `username` to the issuer; "Tokens from one authorization server MUST NOT be associated with an account established with a different issuer, even if the usernames are the same" |
| [Shopify](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/online-access-tokens) | Trust `email` only when `email_verified` is `true` |
| [Yahoo](https://developer.yahoo.com/oauth2/guide/flows_authcode/) | Deprecates the plain member in favour of `id_token` |

Most vendors (Fitbit, Threads, Instagram, Mercado Libre, Polar, Heroku) publish the member with no rationale or warning at all, and none of the vendor pages read discusses the member as an authentication assertion. The adjacent IETF threat text concerns access tokens delivered through the front channel: [RFC 6749 section 10.16](https://www.rfc-editor.org/rfc/rfc6749.html#section-10.16) and [RFC 6819 section 4.4.2.6](https://www.rfc-editor.org/rfc/rfc6819.html#section-4.4.2.6) ("The client assumes that, because it was able to obtain information about the user, the user has been authenticated"). A search of [RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html) found no passage about a user identifier in the token response.

### Conditions and refresh behaviour vary

Returned on code exchange and refresh: Fitbit, Yahoo, VK ID, Mercado Libre, Heroku, FusionAuth. Returned on code exchange only: Strava `athlete`, Instagram `user_id`, WeChat `unionid`. Scope- or mode-gated: IndieAuth `profile`, Shopify `associated_user` (online tokens), Feishu v1 `email` and `mobile`, QQ Connect `openid` (`need_openid=1`), SMART context members. Most other vendors document no condition.

### Some providers have moved away from the pattern

Yahoo deprecated `xoauth_yahoo_guid` for `id_token`; Feishu dropped its v1 profile members in v2; Alipay is migrating from a global `user_id` to a pairwise `open_id`; mailmaint removed `username` in -08. The mailmaint removal was procedural (avoiding new OAuth protocol elements in a non-OAuth WG document) rather than a rejection of the need, according to the list messages cited above, which were read through a summarising fetch.

### Many of these responses are not RFC 6749-shaped

Douyin, Feishu and Withings wrap members in an envelope (`data`, `body`); Instagram uses a `data` array; Xiaomi prefixes `&&&START&&&`; QQ Connect and Bitly default to form encoding; Yammer makes `access_token` an object; Slack nests the user token inside `authed_user`.

### IANA OAuth Parameters registry

| Parameter | Registered usage location | Reference | Token response? |
|---|---|---|---|
| `sub` | authorization request | RFC 7519 section 4.1.2; RFC 9101 | **No** |
| `username` | token request | RFC 6749 | **No**; mailmaint -07's request to add it was withdrawn in -08 |
| `userinfo` | not registered | none | n/a |
| `id_token` | authorization response, access token response | OpenID Connect Core 1.0 | Yes, as a signed JWT |
| `authorization_details` | authorization request, token request, token response | RFC 9396 | Yes; may carry account identifiers |
| `me`, `profile`, `patient`, `fhirUser`, `user_id` | not registered | none | n/a (`fhirUser` and `user_id` checked via summarising fetch) |

Source: [IANA OAuth Parameters CSV](https://www.iana.org/assignments/oauth-parameters/parameters.csv) and [registry page](https://www.iana.org/assignments/oauth-parameters/oauth-parameters.xhtml); the registry's last-updated date of 2026-11-10 comes from a summarising fetch. No registered token response parameter is a plain user identifier. Because `sub` already has a row, registering it for the token response is an update to an existing entry, the same pattern mailmaint -07 used for `username`; `userinfo` is a new registration.

## Checked, does not do this

Identity only inside an `id_token`, or no user member at all. Confirmed from the linked page unless noted.

### Specifications

| Specification | Finding |
|---|---|
| [OpenID Connect (IANA entries)](https://www.iana.org/assignments/oauth-parameters/parameters.csv) | `id_token` and `session_state` only; no plain claims in the token response. OIDC Core itself was not re-fetched |
| [RFC 8693 Token Exchange](https://www.rfc-editor.org/rfc/rfc8693.html#section-2.2.1) | `issued_token_type`; no subject member |
| [draft-ietf-oauth-first-party-apps](https://datatracker.ietf.org/doc/draft-ietf-oauth-first-party-apps/) | Adds only opaque `auth_session` (via summarising fetch) |
| [draft-hunt-oauth-v2-user-a4c-05](https://www.ietf.org/archive/id/draft-hunt-oauth-v2-user-a4c-05.txt) | 2014, expired; used an `id_token` JWT |
| [Solid-OIDC](https://solidproject.org/TR/oidc) | WebID is a claim in the ID Token (via summarising fetch) |
| [RFC 5849 (OAuth 1.0)](https://www.rfc-editor.org/rfc/rfc5849.txt) | Defines no user identifier; vendor additions were extensions |
| [draft-ietf-mailmaint-oauth-public-08](https://www.ietf.org/archive/id/draft-ietf-mailmaint-oauth-public-08.html#section-6.2) | Five standard members only |

### Vendors with identity only in `id_token`

| Vendor | Note |
|---|---|
| [Apple, Sign in with Apple](https://developer.apple.com/documentation/signinwithapplerestapi/tokenresponse) | `access_token`, `expires_in`, `id_token`, `refresh_token`, `token_type` |
| [Google](https://developers.google.com/identity/protocols/oauth2/web-server) | No plain member (page read via summarising fetch); the OIDC page was not fetched, so only the absence is confirmed |
| [Microsoft identity platform](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow) | `id_token` "Only provided if `openid` scope was requested" |
| [Kakao Login](https://developers.kakao.com/docs/latest/en/kakaologin/rest-api) | `id_token` only with OIDC enabled |
| [LINE Login v2.1](https://developers.line.biz/en/reference/line-login/) | `id_token` only with `openid`. One pass listed this from the reference; another could not read the client-rendered page and marked it unconfirmed |
| [Yahoo! JAPAN YConnect v2](https://developer.yahoo.co.jp/yconnect/v2/authorization_code/token.html) | Standard members and `id_token` |
| [Sign in with Vercel](https://vercel.com/docs/sign-in-with-vercel/authorization-server-api) | OIDC-style; no plain user member |
| [Intuit QuickBooks](https://github.com/intuit/oauth-jsclient/blob/master/README.md) | `id_token` only; `realmId` is a redirect parameter (SDK source) |
| [Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/token-endpoint.html), [Okta](https://developer.okta.com/docs/guides/implement-grant-type/authcode/main/), [Zitadel](https://zitadel.com/docs/apis/openidoauth/endpoints) | Standard members and `id_token` |

### Vendors with no user member

| Vendor | Note |
|---|---|
| [Amazon Login with Amazon](https://developer.amazon.com/docs/login-with-amazon/authorization-code-grant.html) | Standard members |
| [Atlassian / Jira Cloud](https://developer.atlassian.com/cloud/jira/platform/oauth-2-3lo-apps/) | `access_token`, `expires_in`, `scope` |
| [Baidu](https://openauth.baidu.com/doc/doc.html) | Adds `session_key`, `session_secret`; no user ID |
| [Basecamp](https://github.com/basecamp/api/blob/master/sections/authentication.md) | Separate `authorization.json` call |
| [Box](https://developer.box.com/reference/post-oauth2-token/) | `restricted_to` is a file/folder restriction |
| [Canva Connect](https://www.canva.dev/docs/connect/api-reference/authentication/generate-access-token/) | Standard members |
| [ClickUp](https://developer.clickup.com/reference/getaccesstoken) | `access_token` only |
| [Coinbase](https://docs.cdp.coinbase.com/coinbase-app/oauth2-integration/integrations) | Standard members |
| [Dailymotion](https://developers.dailymotion.com/guides/authentication/) | Current guide shows no `uid`; a legacy `uid` member is unconfirmed (see below) |
| [Discord (user flow)](https://docs.discord.com/developers/topics/oauth2) | Standard members |
| [DocuSign](https://developers.docusign.com/platform/auth/authcode/authcode-get-token/) | Separate `getUserInfo` call |
| [Facebook Login](https://developers.facebook.com/docs/facebook-login/guides/advanced/manual-flow) | `access_token`, `token_type`, `expires_in` |
| [Fastmail](https://www.fastmail.com/dev/) | Five standard members; clients are pointed to the JMAP session resource (via summarising fetch) |
| [Feishu / Lark v2](https://open.feishu.cn/document/authentication-management/access-token/get-user-access-token) | Dropped v1's profile members |
| [Freesound](https://freesound.org/docs/api/authentication.html) | Standard members |
| [GitHub OAuth apps](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps) | `access_token`, `scope`, `token_type` |
| [GitHub Apps user tokens](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-a-user-access-token-for-a-github-app) | Six members, none identifying the user (via summarising fetch) |
| [GitLab](https://docs.gitlab.com/api/oauth2/) | Standard members plus `created_at` |
| [Gusto](https://docs.gusto.com/app-integrations/docs/oauth2) | Standard members |
| [HubSpot](https://developers.hubspot.com/docs/api-reference/auth-oauth-v1/tokens/post-oauth-v1-token) | User and portal data from a separate token-metadata call |
| [Intercom](https://developers.intercom.com/docs/build-an-integration/learn-more/authentication/setting-up-oauth) | `token_type`, `token`, `access_token` |
| [Kick](https://docs.kick.com/getting-started/generating-tokens-oauth2-flow) | Standard members |
| [Linear](https://linear.app/developers/oauth-2-0-authentication) | Standard members |
| [LinkedIn](https://learn.microsoft.com/en-us/linkedin/shared/authentication/authorization-code-flow) | Standard members |
| [Mail.ru](https://o2.mail.ru/docs/) | `expires_in`, `access_token`, `refresh_token` |
| [MapMyFitness / Under Armour](https://developer.mapmyfitness.com/docs/v71_OAuth_2_Intro/) | Standard members |
| [Mastodon](https://docs.joinmastodon.org/methods/oauth/) | Separate `GET /oauth/userinfo` |
| [Medium](https://github.com/Medium/medium-api-docs) | API archived; standard members |
| [Meetup](https://www.meetup.com/api/authentication/) | Standard members |
| [monday.com](https://developer.monday.com/apps/docs/oauth) | `access_token`, `token_type`, `scope` |
| [Naver Login](https://developers.naver.com/docs/login/api/api.md) | Standard members; one pass saw only code samples, another read the output field list |
| [Oura](https://cloud.ouraring.com/docs/authentication) | Standard members |
| [Patreon](https://docs.patreon.com/) | Standard members |
| [PayPal REST](https://developer.paypal.com/api/rest/authentication/) | `app_id` identifies the app (client credentials example) |
| [Pinterest API v5](https://developers.pinterest.com/docs/getting-started/authentication-and-scopes/) | Standard members |
| [Reddit](https://github.com/reddit-archive/reddit/wiki/OAuth2) | Standard members |
| [Roblox](https://create.roblox.com/docs/cloud/auth/oauth2-reference) | Standard members in the example shown |
| [Snapchat Marketing API](https://developers.snap.com/api/marketing-api/Ads-API/authentication), [Login Kit](https://developers.snap.com/snap-kit/login-kit/Tutorials/web) | Standard members |
| [SoundCloud](https://developers.soundcloud.com/docs/api/guide) | Standard members |
| [Spotify](https://developer.spotify.com/documentation/web-api/tutorials/code-flow) | Standard members |
| [Squarespace](https://developers.squarespace.com/oauth) | `token_type`, `access_token`, `access_token_expires_at` |
| [Stack Exchange](https://api.stackexchange.com/docs/authentication) | Form-encoded `access_token` and `expires` |
| [Todoist](https://developer.todoist.com/guides/) | Standard members |
| [Tumblr OAuth 2](https://github.com/tumblr/docs/blob/master/api.md) | `id_token` shown as `false` |
| [Twitch](https://dev.twitch.tv/docs/authentication/getting-tokens-oauth/) | `user_id` and `login` come from the separate [`/validate`](https://dev.twitch.tv/docs/authentication/validate-tokens/) endpoint |
| [Typeform](https://www.typeform.com/developers/get-started/applications/) | Standard members |
| [Uber Riders API](https://developer.uber.com/docs/riders/guides/authentication/user-access-token) | Standard members |
| [WHOOP](https://developer.whoop.com/docs/developing/oauth/) | Standard members |
| [X / Twitter OAuth 2.0](https://github.com/xdevplatform/twitter-api-typescript-sdk/blob/main/src/OAuth2User.ts) | **SDK type only**: five members, none identifying the user. The [docs](https://docs.x.com/fundamentals/authentication/oauth-2-0/user-access-token) show no response body; an SDK type cannot rule out extra members |
| [Yandex ID](https://yandex.com/dev/id/doc/en/codes/code-url) | Standard members |
| [Zendesk](https://developer.zendesk.com/api-reference/ticketing/oauth/grant_type_tokens/) | Standard members |
| Zoho, Wrike, Pipedrive, Zoom | Routing members only; links in the context-identifier section |

## Unconfirmed or not checked

Nothing in this section should be cited as fact. Member names come from third-party sources or prior knowledge, not from a source that was read.

### Reported to return a member, not confirmed from an official source

| Provider | Reported member(s) | Why unconfirmed |
|---|---|---|
| Battle.net | `sub` (would be the closest vendor match to the draft's name) | Docs are JavaScript-only: https://develop.battle.net/documentation/guides/using-oauth/authorization-code-flow |
| Epic Games | `account_id` | Docs returned an empty shell (https://dev.epicgames.com/docs/web-api-ref/authentication); search-engine snippet and third-party sources only |
| Imgur | `account_id`, `account_username` | https://apidocs.imgur.com/ not fetchable; the [vendor Python client](https://github.com/Imgur/imgurpython/blob/master/imgurpython/client.py) does not model them; third-party libraries only |
| Vercel integrations | `user_id` | Example response collapsed on the [page](https://vercel.com/docs/rest-api/vercel-api-integrations); only `team_id` and `installation_id` confirmed |
| VK legacy API (`oauth.vk.com`) | `user_id`, `email` with the `email` scope | `dev.vk.com` could not be fetched: https://dev.vk.com/ru/api/access-token/authcode-flow-user |
| Dailymotion (legacy) | `uid` | Current guide does not show it; archive fetch was rate-limited |
| Instagram Basic Display API | `user_id` | API shut down December 2024; old reference URLs redirect |
| DingTalk | `corpId` | Page yielded no content: https://open.dingtalk.com/document/orgapp/obtain-user-token |
| Oracle Health (Cerner) | `patient`, `encounter`, `user`, `tenant`, historically `username` | Redirect target not read: https://docs.oracle.com/en/industries/health/millennium-platform-apis/mfrap/authorize.html |
| Razorpay | `razorpay_account_id`, `public_token` | Docs URL returned 404 |
| GoCardless | `organisation_id`, `email` | Page fetched did not contain it |
| LiveChat / Text | `account_id`, `organization_id` | Example not found |
| Ecwid / Lightspeed, MYOB | `store_id`; `user` object | Not checked |
| Salesforce community members | `sfdc_community_url`, `sfdc_community_id` | SDK constants exist; presence in responses not confirmed |
| Stalwart Mail Server | `username` (as a mailmaint implementer) | [Docs](https://stalw.art/docs/auth/oauth/overview) say it implements the profile but show no token response |
| Feishu / Lark v3 | unknown | `accounts.feishu.cn/oauth/v3/token` page not read |
| Withings | `userid` type and whether always present | Member name confirmed only; see the identifier table |

### Fetched, token response not found

Xero, Airtable, Webex, Bitbucket Cloud, Mailchimp, Webflow, Amazon Selling Partner API, FreshBooks, ServiceNow, Gumroad, eBay (HTTP 403), Wahoo, Untappd, Eventbrite, Deezer, Foursquare, Garmin, Trello (OAuth 1.0a), Pocket, Last.fm, Paytm, Authentik, node-oidc-provider (only `extraTokenClaims` found in its [README](https://github.com/panva/node-oidc-provider/blob/main/docs/README.md); not a definitive no).

### Not reached

Vendors: Netlify, Smartsheet, Dropbox Sign, Twilio, Freshworks, Workday, Egnyte, Adobe, Autodesk, Wix, WooCommerce, Deel, Steam, Unity, Discourse, Lyft, Venmo, Peloton, Telegram, Huawei Account Kit, Rakuten, Mercari, Odnoklassniki, Zalo, Grab, Gojek, Truecaller, athenahealth, Allscripts / Veradigm, AOL, GMX / web.de, Proton, iCloud.

Government, telecom and banking: FranceConnect, itsme, SPID / CIE, Singpass / MyInfo, BankID variants, Vipps, Orange, Deutsche Telekom, newer DigiLocker / API Setu documentation, TrueLayer, Tink, Klarna, UK Open Banking, Berlin Group NextGenPSD2, Brazilian Open Finance.

Identity products: WSO2 Identity Server, Gluu, PingFederate / PingOne, Laravel Passport's own docs, Auth0 Rules.

Specifications: OpenID Connect Native SSO, CIBA, OpenID Federation, HEART, Mobile Connect, FAPI 1.0 / 2.0, the current OAuth 2.1 draft text on authentication, draft-ietf-mailmaint-pacc. No older OAuth WG thread proposing a plain user identifier in the token response was found, from limited searching of an archive that is hard to search by keyword.

## Conclusion

The catalog shows the same design invented independently many times, and under almost as many names. That is the practical argument for registering one: the deployed members split cleanly into a bare identifier and a small claims object, which are the two shapes the draft defines. The only place the draft's chosen name already exists is AT Protocol, and its `sub` is a globally resolvable DID with mandatory issuer verification, so the draft's rules on when `sub` appears and how far a client may trust it are the text most likely to be read against an existing deployment.

Two things the survey does not settle are worth keeping in view. First, the mailmaint use case wanted a human-recognisable login name, which an opaque `sub` does not provide; whether a `userinfo` object satisfies that group is untested, since they moved to a separate endpoint instead. Second, a sizeable share of the confirmed vendor members identify a tenant, merchant or patient rather than a person, and multi-tenant platforms tend to return both, so implementers adopting `sub` and `userinfo` will keep their context members alongside them.
