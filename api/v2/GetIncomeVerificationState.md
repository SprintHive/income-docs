## Fetching the income verification state

The following endpoint can be used to fetch the current state of an income verification request.

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)

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
  "subStatus": "HIGH_INCOME_CONFIDENCE",
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

The following table explains the status and subStatus in the example response above. 

| Status            | SubStatus                | Description                                                                                                   |
|-------------------|--------------------------|:--------------------------------------------------------------------------------------------------------------|
| IN_PROGRESS       | WAITING_FOR_DOCUMENTS    | Waiting for documents to determine income                                                                     |
| IN_PROGRESS       | PROBLEMS_WITH_DOCUMENTS  | One or more documents have been received that has failed business rules e.g. document is too old              |
| IN_PROGRESS       | DATA_EXTRACTION_FAILED   | One or more documents have failed data extraction                                                             |
| IN_PROGRESS       | NO_INCOME_DETECTED       | Enough documents were received to try and detect income but income could not be found                         |
| IN_PROGRESS       | LOW_INCOME_CONFIDENCE    | Income was found but the system is not confident with the answer                                              |
| REFERRED_TO_FRAUD | SUSPECTED_DOCUMENT_FRAUD | One or more documents has failed a fraud check                                                                |
| REFERRED_TO_FRAUD | SUSPECTED_CASE_FRAUD     | An agent or third party system has suspected fraud                                                            |
| SUCCESS           | HIGH_INCOME_CONFIDENCE   | Income was successfully detected and the system is confident with its answer                                  |
| SUCCESS           | MANUALLY_CAPTURED        | Income was successfully captured by an agent or third party system                                            |
| SUCCESS           | LOW_INCOME_CONFIDENCE    | Income was successfully detected but the system is not confident with the answer                              |
| FAILED            | WAITING_FOR_DOCUMENTS    | Income was not detected because we did not receive enough documents within the time allowed                   |
| FAILED            | PROBLEMS_WITH_DOCUMENTS  | Income was not detected because the business rules were not met                                               |
| FAILED            | DATA_EXTRACTION_FAILED   | Income was not detected because the was a problem with extracting the data                                    |
| FAILED            | NO_INCOME_DETECTED       | Enough documents were received to try and detect income but income could not be found within the time allowed |
| CONFIRMED_FRAUD   | CONFIRMED_DOCUMENT_FRAUD | At least one document has failed a failed a check that is classified as high risk                             |
| CONFIRMED_FRAUD   | MANUAL_CONFIRMED_FRAUD   | Either an agent or third party system has confirmed                                                           |
| CONFIRMED_FRAUD   | CONFIRMED_CASE_FRAUD     | The case has failed a failed a check that is classified as high risk                                          |

### How to route the applications based on status

When the status is SUCCESS and result.status is "HIGH_CONFIDENCE" the income has been determined so the application can proceed to the next step in your process.  
When the status is CONFIRMED_FRAUD the application should be sent to the fraud department for review.  
When the status is FAILED the application can be sent for manual review.  
