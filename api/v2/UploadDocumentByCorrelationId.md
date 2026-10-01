## Uploading a document using the correlationId

If you have the `incomeVerificationId`, [upload the document directly to the case](./UploadDocument.md) instead.
The `incomeVerificationId` should be stored alongside the `correlationId` when the case is created, for easy reference.

The following endpoint can be used to upload a document using the `correlationId` you supplied when 
[creating the income verification case](./CreateIncomeVerificationRequest.md#correlationid). This is only an option if you use the mailbox management system and there 
is an open document linking request for that income case.
(Mailbox management is where customers email their documents to a monitored mailbox. The documents are then 
automatically linked to the relevant income verification.)

The document is linked to every active case that has the given `correlationId`. A single file can be uploaded at a time.
To upload multiple files, make a separate api post for each file. A unique `documentId` will be returned in the response
for each document uploaded.

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)
* Your token has the document linker `document:upload` scope

Endpoint: ```document-linker/v1/document```  
Method: POST  
Body type: multipart/form-data

Body:

| correlationId | test-1                   |
|---------------|:-------------------------|
| file          | exampleBankStatement.pdf |

Response:
- HTTP status code: 200

```json
{
  "documentId": "a26b2ffa-146e-4fe4-a526-8083851eecef"
}
```

- HTTP status code: 404

No active case was found for the `correlationId`. This happens when the `correlationId` does not match a case, or when
the case is no longer accepting documents.

```json
{
  "requestError": {
    "errorType": "document-request-not-found",
    "httpCode": 404,
    "timestamp": "2026-10-01T06:50:48.651944820Z",
    "correlationId": "test-1",
    "message": "No active linking request found for correlationId: test-1"
  }
}
```

## Errors

This endpoint is served by the document linker, so its errors use a different envelope from the income endpoints.
The error is returned under `requestError` instead of `error`, it includes a `message` and the `correlationId`
that was sent, and it has no `traceId` or `errors` array. If you reuse error handling written for the income
endpoints, make sure it also reads `requestError`.

The `401` and `403` responses that apply to every call are described in the
[security section](../../guides/security/CreatingJsonWebToken.md).