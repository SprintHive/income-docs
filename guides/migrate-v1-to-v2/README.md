# Migrating from v1 to v2

The purpose of this guide is to explain how to migrate from v1 to v2

## Prerequisites
* You have read and understand the security section [click here for more details](../security/CreatingJsonWebToken.md)

## Creating an income verification request

 * The structure of the payload has changed to support the declaration of primary and other income
 * physicalEntity has been renamed to applicantDetails

Endpoint: ```v2/incomeVerification```  
Method: POST  
Body:

Before:
```json
{
  "correlationId": "some-uuid",
  "declaration": {
    "nettIncome": 43000,
    "grossIncome": 5000
  },
  "physicalEntity": {
    "idNumber": "1234567890",
    "firstName": "Joe",
    "lastName": "Soap",
    "bankName": "Nedbank",
    "bankAccountNumber": "1180234567",
    ...
  },
  "config": {
    ...
  }
}
```

After:
```json
{
  "correlationId": "some-uuid",
  "primaryIncome": {
    "nettIncome": 43000,
    "grossIncome": 5000
  },
  "applicantDetails": {
    "idNumber": "1234567890",
    "firstName": "Joe",
    "lastName": "Soap",
    "bankName": "Nedbank",
    "bankAccountNumber": "1180234567",
    ...
  },
  "config": {
    ...
  }
}
```

## Uploading a document

The payload is the same as v1 the only change is the endpoint path.

Before: ```"/v1/incomeVerification/{incomeVerificationId}/upload/file"```  
After: ```v2/incomeVerification/{incomeVerificationId}/document```

> See the quick start guide for more info

## Fetching the income verification state

 * The statuses have been simplified by not including end states.
 * Fetching the state of the income verification xxx

Endpoint: ```v2/incomeVerification/${incomeVerificationId}/state```  
Method: GET  
Response:
```json
{
  "status": "SUCCESS",
  "documents": [
    {
      "documentId": "d3107619-26d9-4093-a7e4-79fd9d4caeb2",
      "documentHash": "36420742ea5a848249bac915e7ba3c951cdda395",
      "type": "BANK_STATEMENT",
      "received": "2022-08-11T10:20:30.478",
      "status": "DATA_EXTRACTION_COMPLETED"
    }
  ],
  "result": {
    "method": "AUTOMATED",
    "status": "HIGH_CONFIDENCE",
    "accountHolderName": "Mr Raoul McClureBee",
    "bankName": "Nedbank",
    "bankAccountNumber": "1180234567",
    "analysisResult": {
      "primaryIncome": {
        "declaredNettIncome": 4500,
        "detectedNettIncome": 4238.22,
        "confidence": 0.9811,
        "variance": -0.06177,
        "payments": [
          {
            "date": "2021-06-29",
            "amount": 4238.22
          }
        ],
        "payDay": 29,
        "payCycle": 30
      }
    }
  }
}
```

### What do the different Statuses mean?

For more information about the statuses that are available in V2 [see](../../api/v2/GetIncomeVerificationState.md)

### When will the web hook notification be sent?

For more information about the notifications please [see](../notifications/Notifications.md)