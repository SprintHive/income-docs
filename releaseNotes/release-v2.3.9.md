# Release Notes for v2.3.9

Date: 1 March 2024

## Introducing a new document status CLASSIFICATION_FAILED

When a document cannot be classified we would like to send notifications so that the case can be routed in accordance to the partners requirements.

For example:
- Send the case to manual review
- Trigger the chase process

When a document cannot be classified the document status will be CLASSIFICATION_FAILED with the document type of "UNKNOWN"
When a document fails classification the income verification request status will be IN_PROGRESS with the sub-status PROBLEMS_WITH_DOCUMENTS
The change of the sub-status will result in notifications being sent if enabled.

An example of the response from the get state endpoint:  ```income/v2/incomeVerification/${incomeVerificationId}/state```


```json
{
    "status": "IN_PROGRESS",
    "subStatus": "PROBLEMS_WITH_DOCUMENTS",
    "documents": [
        {
            "documentId": "c833998a-d261-477f-a664-68e952347b75",
            "documentHash": "5ab2f8fe9d1f29e816905aad012df82199cfc643",
            "type": "UNKNOWN",
            "received": "2024-03-01T07:03:46.436",
            "status": "CLASSIFICATION_FAILED"
        }
    ],
    ...
}
```

## Introducing a new document type MULTI_DOC

When a document is received and it contains more than one type of document, a payslip and a bank statement for example, the system will split the document up into multiple documents grouped by type.
The original document type will be MULTI_DOC instead of UNKNOWN

An example of the response from the get state endpoint:  ```income/v2/incomeVerification/${incomeVerificationId}/state```

```json
{
    "status": "SUCCESS",
    "subStatus": "HIGH_INCOME_CONFIDENCE",
    "documents": [
        {
            "documentId": "cd793003-a784-4863-92c5-c8963e5f15f6",
            "documentHash": "23a334019d8024583e9d4079ad10ee4feba464c4",
            "type": "MULTI_DOC",
            "received": "2024-03-01T06:55:24.408",
            "status": "RECEIVED"
        },
        {
            "documentId": "e296d737-d300-4e32-a4e0-2fb6fe93e1a5",
            "documentHash": "24f98fd2df53ab9420a61da0aa44c0ad21d212ab",
            "type": "BANK_STATEMENT",
            "received": "2024-03-01T06:55:25.168",
            "status": "DATA_EXTRACTION_COMPLETED"
        },
        {
            "documentId": "9eaef9f7-d9de-4169-a75c-67da8b7d38fb",
            "documentHash": "9a3f78f52e1a28208d99b0138cdbc4d101811768",
            "type": "PAYSLIP",
            "received": "2024-03-01T06:55:25.291",
            "status": "TOO_OLD"
        }
    ],
...
}
```