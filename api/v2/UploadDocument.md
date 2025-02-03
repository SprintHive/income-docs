## Uploading a document to an income verification case

The following endpoint can be used to upload a document to an income verification case. A single file can be uploaded at
a time. To upload multiple files to one case, make a separate api post for each file using the `{incomeVerificationId}`
for the income verification case. A unique `documentId` will be returned in the response for each document uploaded.

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)

Endpoint: ```income/v2/incomeVerification/{incomeVerificationId}/document```  
Method: POST  
Body type: multipart/form-data

Body:

| file | exampleBankStatement.pdf                                                                                                     |
|------|:--------------------------------------------------------------------------------------------------------------------------|


Response:
- HTTP status code: 200

```json
{
  "documentId": "a26b2ffg-146e-4fe4-a526-8083851eecef"
}
```

- HTTP status code: 404
