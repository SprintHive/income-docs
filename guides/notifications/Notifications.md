# Notifications

There are two different notifications available namely Document Processed and Status Changed notifications.

## The process

![notification-sequence-diagram](images/document-processed-notification-diagram.png)


## Document Processed Notifications

The document processed notifications can be useful to keep an external system in sync when documents are sent directly
to SprintHive via the email inbox or via the upload site. 

Your system: This is the system that integrates with SprintHives income verification product.
SprintHive: This is SprintHive's income verification product.

### When does the document processed event fire?

* When extracting a document fails
* When document validation fails
* When a document cannot be linked to a matched to the applicant (declared data is required)
* When a document has been successfully extracted

## Status Changed Notifications

The status changed notification is typically used to route the application to an automated or a manual review flow

### When does the status changed event fire?

The status changed event is fired when an income verification requests status changes as a result of documents received.
The most common scenario is for the status to change from IN_PROGRESS to SUCCESS or FAILED


## Creating the Web hook 

To enable the ability to receive a webhook notification your system needs to expose and endpoint which can receive
a POST message with a JSON payload.

An example of the notification payload
```json
{
  "incomeVerificationId": "",
  "correlationId": "",
  "documentId": ""
}
```

Your system can use the incomeVerificationId to fetch the latest state ([for more info](../../api/GetIncomeVerificationState.md)).
The response has a list of documents that are linked to the income verification request and can be used to download the document ([for more info](../../api/GetDocumentContent.md))
and then upload it to your system's file storage. 

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
    "accountHolderName": "Rev Thomas Bayes",
    "bankName": "Capitec Bank",
    "bankAccountNumber": "1234567890",
    "analysisResult": {
      "primaryIncome": {
        "declaredNettIncome": 20000,
        "detectedNettIncome": 19226.92,
        "confidence": 0.9892,
        "variance": -0.04021,
        "payDates": [
          {
            "date": "2021-05-01",
            "amount": 19226.92
          },
          {
            "date": "2021-04-28",
            "amount": 19226.92
          }
        ],
        "payDay": 28,
        "payCycle": 30
      }
    }
  }
}
```