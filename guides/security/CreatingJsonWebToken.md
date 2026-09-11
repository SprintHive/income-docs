# Creating a Json Web Token (JWT)

## Prerequisites

As part of the onboarding process you should have received an email from SprintHive with the following details:

* API key
* Client Id
* Client Secret
* Authentication Endpoint
* Audience

## API Key

Your system integrating with these environments will require a system API key for each environment. 
The API key should be included in an HTTP header called apikey on all API requests.

## Calling the authentication endpoint

An example of what the curl command:
```shell
curl --location --request POST '<authentication_endpoint>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'client_id=<client_id>' \
--data-urlencode 'client_secret=<client_secret>' \
--data-urlencode 'audience=<audience>'
```

This request returns an access_token which must be added to the Authorization header of API requests 
after the Bearer prefix. E.g. Authorization: Bearer <access_token>. 

The response also contains an expires_in field that reports the number of seconds for which the access_token is valid.
It is *critically* important to cache the access_token for this period of time before a new token is obtained;  
the number of tokens you can request has a limit which will likely be breached if the token is not cached.

> NB: A high volume of new token requests will be treated as a high 
> severity incident for SprintHive and may result in new token requests from being blocked.

## Errors

Every request carries two credentials - the `apikey` header and the `Authorization: Bearer` token - and each
one can be rejected on its own. A rejected token gives you a `401` in the same error envelope used everywhere
else in the API. A rejected API key gives you a `403` in a different shape, because it is refused before the
request ever reaches the Income API.

### 401 - the token was rejected

```json
{
  "error": {
    "errorType": "auth-token-invalid",
    "httpCode": 401,
    "traceId": "xxxxxx-xxxxxx-xxxxx-xxxxx",
    "errors": [
      {
        "field": "Authentication",
        "errorCode": "Invalid or missing JWT"
      }
    ],
    "timestamp": "2020-01-01T00:00:00.0000000Z"
  }
}
```

There are two `errorType` values you can see on a `401`:

| `errorType`          | Meaning                                                                                                                                                    |
|----------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `auth-token-invalid` | A token was sent and rejected - it has **expired**, or it is malformed, incorrectly signed, issued by an unknown authority, or carries the wrong audience. |
| `not-authenticated`  | No `Authorization` header was sent at all, or a valid token was used against the wrong environment for its audience.                                       |

The `errors` array is the same fixed entry - `Authentication` / `Invalid or missing JWT` - whichever of the two
you get.

**An expired token is reported as `auth-token-invalid`, exactly like a corrupt one.** Expiry is not signalled
separately, so do not write code that waits for an "expired" code - it will never arrive. Treat any `401` the
same way: request one fresh token, retry the call once, and fail the request if the second attempt is also
rejected. In practice a `401` on a previously working integration almost always means the cached
`access_token` outlived its `expires_in`.

Note that no `WWW-Authenticate` header is returned with a `401`. HTTP clients that rely on that header to
trigger an automatic re-authentication will not do so here; your own code has to handle the retry.

### 403 - the request was not allowed

A `403` means your credentials were understood but the call was not permitted. The two causes are told apart
by the body.

The `apikey` header is missing or is not valid for the environment you are calling:

```json
{
  "message": "You cannot consume this service"
}
```

The `apikey` was accepted and the token is valid, but the token does not carry the scope the endpoint
requires:

```json
{
  "error": {
    "errorType": "not-authorized",
    "httpCode": 403,
    "traceId": "xxxxxx-xxxxxx-xxxxx-xxxxx",
    "errors": [],
    "timestamp": "2020-01-01T00:00:00.0000000Z"
  }
}
```

| Response                                         | What to check                                                                                                          |
|--------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------|
| `{"message": "You cannot consume this service"}` | The `apikey` header. It is either absent, or it belongs to a different environment than the one you are calling.       |
| `not-authorized 403`                             | The scopes granted to your client. The `apikey` and the token were both fine; the token just lacks the required scope. |

The quickest way to tell them apart in code is to look for the `error` object: the API key rejection does not
have one.

On a `not-authorized` the `errors` array is **always empty** - the response will not tell you which scope was
missing. Work it out from the table below, using the endpoint you called.

### Scopes

Your client is granted a set of scopes when it is onboarded, and each endpoint requires one of them. If you
need a scope your client does not currently have, contact SprintHive.

| Scope                                     | Grants                                                                       |
|-------------------------------------------|:-----------------------------------------------------------------------------|
| `incomeVerification:create`               | Create Income Verification cases and read income detector strategies         |
| `incomeVerification:read`                 | Read Income Verification cases                                               |
| `incomeVerification:state:read`           | Read the most recent state for an income verification                        |
| `incomeVerification:file:upload`          | Upload documents to an income verification                                   |
| `incomeVerification:bankStatement:upload` | Upload extracted bank statement data                                         |
| `incomeVerification:manualCapture:create` | Capture the nett income for an Income Verification                           |
| `incomeVerification:fraud:create`         | Suspect, and confirm fraud                                                   |
| `incomeVerification:document:read`        | Read the state for a document                                                |
| `incomeVerification:confirmation:update`  | Record the confirmation result for an Income Verification case               |
| `incomeVerification:filter:read`          | Filter Income Verification cases                                             |
| `incomeVerification:history:read`         | Read the history timeline of income verification cases                       |
| `incomeVerification:documentChase:read`   | Read the latest state of outstanding documents for income verification cases |
| `incomeVerification:upload`               | Create an upload request                                                     |
| `incomeVerification:indicator`            | Read the indicators for an income verification                               |

The scope each endpoint requires is declared against that endpoint in the
[OpenAPI spec](../../openApi/openapi-public-v2.yaml).

### Recommendations

* On a `401`, fetch a new token and retry the call once. Do not branch on the `errorType` - the causes are
  not distinguishable and the remedy is the same for all of them.
* Do not retry a `403`. Neither an API key nor a missing scope will start working on its own, and the retry
  will fail in exactly the same way.
* Keep caching the `access_token` for its `expires_in` even while handling `401`s. Fetching a new token on
  every request, or on every `401`, is the behaviour described above that gets integrations blocked.
* Quote the `traceId` when reporting a problem to SprintHive. It is absent from some responses, so read it
  defensively rather than requiring it.
* Match on `errorType` rather than on any message text, and treat the set of values as one that may grow.

