# Data retention and deletion

Income verification cases are not kept forever. Once a case falls outside the configured data retention period it is
deleted, along with the data extracted from the documents attached to it. This guide explains when that happens and,
more importantly, what your system sees when it calls the API for a case that has been deleted.

## Prerequisites
* You have read and understand the security section [click here for more details](../security/CreatingJsonWebToken.md)

## The retention policy

Data retention is controlled by tenant configuration, so the exact values that apply to you are agreed with SprintHive
during onboarding. Contact SprintHive if you need to confirm or change the settings on your tenant.

| Setting            | Description                                                                                                                         | Default  |
|--------------------|:------------------------------------------------------------------------------------------------------------------------------------|:---------|
| Deletion enabled   | Whether income cases are deleted at all. When this is disabled nothing is ever deleted and you will never receive a `410` response. | Disabled |
| Automatic deletion | Whether a scheduled job deletes out of retention data every night without anyone asking for it.                                     | Disabled |
| Retention period   | How long an income case is kept, measured from the date the case was created.                                                       | 5 years  |

A case is "out of retention" when its creation date is older than the retention period. Documents are evaluated
separately, using the date the document was received rather than the date the case was created.

## When data is deleted

There are two ways a delete is started, and both delete exactly the same data:

* **Automatically** - when automatic deletion is enabled, a scheduled job runs each night and deletes everything that
  has fallen outside the retention period since the previous run.
* **On request** - SprintHive can delete a single case, or every case currently out of retention, on request. This is
  done through an internal API and is not something your system calls directly.

A delete that fails part way through is retried, and the case is not considered deleted until the delete has
completed. From your system's point of view this distinction does not matter: **the case becomes unavailable the
moment the delete starts**, not when it finishes.

Data that is still inside the retention period cannot be deleted. A request to delete a case that is not yet out of
retention is rejected rather than honoured.

## What is deleted

* The income case, including the declared income, applicant details, configuration and the detected income result.
* The extracted data for every document linked to the case - the bank statement transactions, payslip earnings and
  deductions, and the validation and fraud analysis results.
* The case's entry in the search index, so the case no longer appears in the results of the
  [filter endpoint](../../api/v2/IncomeFiltering.md).
* The `correlationId` reservation used for idempotent creates. See [Reusing a correlationId](#reusing-a-correlationid)
  below.

Summary metrics about the case are retained so that SprintHive can continue to report on volumes and accuracy -
timings, statuses, document counts, the configuration the case ran with and the income figures. They do not contain
the applicant's personal details, the uploaded documents, or the extracted transactions.

The raw file you uploaded is served by a different endpoint to the rest of the API
([Downloading a document](../../api/v2/GetDocumentContent.md)) and is not governed by the policy described here.
Contact SprintHive if you need to know how long the stored files themselves are kept.

## What your system sees after a case is deleted

Calls for a deleted case fail with an `entity-gone 410` error:

```json
{
  "error": {
    "errorType": "entity-gone",
    "httpCode": 410,
    "traceId": "xxxxxx-xxxxxx-xxxxx-xxxxx",
    "errors": [],
    "timestamp": "2020-01-01T00:00:00.0000000Z"
  }
}
```

Note the difference between this and a `404`:

| Response                            | Meaning                                                                                                          |
|-------------------------------------|:-----------------------------------------------------------------------------------------------------------------|
| `entity-gone 410`                   | The `incomeVerificationId` was valid and the case has been deleted. It will never come back.                     |
| `income-verification-not-found 404` | The `incomeVerificationId` is not known to SprintHive - most likely a typo or an id belonging to another tenant. |

Retrying a `410` will never succeed, so your system should treat it as a permanent answer and stop polling the case.

### Which endpoints return 410

Every endpoint scoped to a single income verification returns `410` once the case has been deleted, whether it reads
data or writes it.

| Endpoint                                                                          | Method | Behaviour after the case is deleted                         |
|-----------------------------------------------------------------------------------|:-------|:------------------------------------------------------------|
| `income/v2/incomeVerification`                                                    | POST   | Unaffected - creates a new case                             |
| `income/v2/incomeVerification/${incomeVerificationId}`                            | GET    | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/state`                      | GET    | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/deaConsent`                 | POST   | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/document`                   | POST   | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/document/${documentId}`     | GET    | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/documentChase`              | GET    | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/extractedBankStatements`    | POST   | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/transactions`               | GET    | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/transactions/download/csv`  | GET    | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/enrichedTransactions`       | GET    | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/indicators`                 | GET    | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/fraudAnalysis`              | GET    | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/suspectedFraud`             | POST   | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/confirmFraud`               | POST   | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/confirmNotFraud`            | POST   | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/manualCapture/manualIncome` | POST   | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/confirmation`               | POST   | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/upload`                     | POST   | `entity-gone 410`                                           |
| `income/v2/incomeVerification/${incomeVerificationId}/history`                    | GET    | `200` with an empty list of timeline items                  |
| `income/v2/incomeVerification/filter`                                             | POST   | `200` - deleted cases are no longer returned in the results |

### Deleted documents on a live case

Documents age out of retention on their own, using the date the document was received. A document that is old enough
can therefore be deleted while the case it belongs to is still inside the retention period. When that happens the case
endpoints keep working and the document is still listed in the case state, but fetching that specific document returns
`entity-gone 410`.

### Reusing a correlationId

When your tenant has `idempotentOnCreate` enabled, the `correlationId` of a live case is reserved and a second
create using it is treated as a repeat of the original - see
[correlationId](../../api/v2/CreateIncomeVerificationRequest.md#correlationid) for the detail. Deleting the case
releases that reservation, so the same `correlationId` can be used again for a brand new income verification.

## Recommendations

* Treat SprintHive as the system of record only for the life of the case. If you need the income result, the extracted
  transactions or the uploaded documents for longer than the retention period, copy them into your own storage while
  the case is live. The [notifications guide](../notifications/Notifications.md) describes how to be told when a
  document has been processed so you can fetch and store it as it arrives.
* Handle `410` explicitly wherever you handle `404`. A deleted case is a normal, expected outcome for an old
  `incomeVerificationId`, not an error in your integration.
* Do not retry on `410`, and stop any polling or scheduled reconciliation for that `incomeVerificationId`.
