# Migrating from v1 to v2

The purpose of this guide is to explain how to migrate from v1 to v2

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
| Status            | Description                                                                                                                                                                                                                          |
|-------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| IN_PROGRESS       | Waiting for documents to determine income                                                                                                                                                                                            |
| REFERRED_TO_FRAUD | The income verification request has been suspected of fraud                                                                                                                                                                          |
| SUCCESS           | The income verification request has been processed                                                                                                                                                                                   |
| FAILED            | The income request can fail for the following reasons<ul><li>Not enough documents have been uploaded to meet the business requirements</li><li>Income was detected but was outside the tolerance (confidence is below 95%)</li></ul> |
| CONFIRMED_FRAUD   | Either an agent has confirmed the income verification to be fraudulent or the system has detected high risk fraud                                                                                                                    |

### When will the web hook notification be sent?

The web hook will be notified when the income verification reaches SUCCESS, FAILED or CONFIRMED_FRAUD state.

The message that is posted by the web hook has not been change from v1 to v2 but getting the result payload has changed. 
See the ```v2/incomeVerification/${incomeVerificationId}/state``` endpoint for more info. 
```json
{
  "incomeVerificationId": "some-uuid",
  "correlationId": "some-uuid"
}
```

### How to know if an income has been successfully detected

When the status is SUCCESS and result.status is "HIGH_CONFIDENCE"

> Any other result can be sent for manual review
