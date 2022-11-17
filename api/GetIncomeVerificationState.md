## Fetching the income verification state

The following endpoint can be used to fetch the current state of an income verification request.

This can be used for the following reasons:
* what the result of the income verification request
* get all the documents linked to the income verification request
* get the detected nett income

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}/state```  
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

### How to route the applications based on status

When the status is SUCCESS and result.status is "HIGH_CONFIDENCE" the income has been determined so the application can proceed to the next step in your process
When the status is CONFIRMED_FRAUD the application should be sent to the fraud department for review
When the status is FAILED the application can be sent for manual review
