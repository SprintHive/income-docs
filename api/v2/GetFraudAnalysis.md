# Get Fraud Analysis

The following endpoint can be used to fetch the fraud analysis of an income verification case by its `incomeVerificationId`.
It returns two lists:

* `history` — every fraud decision made on the case, in the order it was made. This covers the automated document
  tampering check run on each document, and any fraud decision captured by an agent or third party system.
* `profileAnalysis` — the results of the applicant profile building analysis, such as the spending curve check.

Use this endpoint to see *why* a case or document was referred to fraud. The case status and the document statuses
themselves are returned by the [income verification state](GetIncomeVerificationState.md) endpoint.

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)
* Your token has the `incomeVerification:read` scope

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}/fraudAnalysis```  
Method: GET  

Below is an example response for a case with three documents:

1. The first document raised a high risk hint, but its overall result was a warning, so it was suspected of fraud.
2. The second document was not sent for tampering checks, so nothing was flagged and no checks are listed.
3. The third document raised three high risk hints and its overall result was high risk, so it was confirmed as fraud.
4. An agent then reviewed the case, first suspecting and then confirming fraud.

The profile analysis ran once on the bank statement transactions and found a low risk spending curve.

Response:
```json
{
  "history": [
    {
      "createdDate": "2026-10-28T14:58:42.512",
      "documentId": "05f4cd89-db51-4c5f-9671-18366375b8f4",
      "decision": "AUTOMATED_SUSPECTED_FRAUD",
      "reason": "System automated document tampering checks",
      "fraudHints": [
        {
          "hintId": "unusual-document-structure",
          "displayName": "Unusual document structure",
          "riskCategory": "HIGH"
        }
      ],
      "completedFraudChecks": [
        "graphics-manipulations",
        "unusual-document-structure",
        "text-manipulations",
        "unusual-document-metadata",
        "editing-tool-usage"
      ]
    },
    {
      "createdDate": "2026-10-28T14:59:45.43",
      "documentId": "3b191184-4b08-41fb-8bac-cb430bd2977a",
      "decision": "AUTOMATED_NOTHING_SUSPICIOUS_DETECTED",
      "reason": "System automated document tampering checks",
      "fraudHints": [],
      "completedFraudChecks": []
    },
    {
      "createdDate": "2026-10-28T15:02:13.721",
      "documentId": "8c231c1e-6949-46ca-acb8-f0bb0d7e55a0",
      "decision": "AUTOMATED_CONFIRMED_FRAUD",
      "reason": "System automated document tampering checks",
      "fraudHints": [
        {
          "hintId": "unusual-document-structure",
          "displayName": "Unusual document structure",
          "riskCategory": "HIGH"
        },
        {
          "hintId": "text-manipulations",
          "displayName": "Text manipulations",
          "riskCategory": "HIGH"
        },
        {
          "hintId": "graphics-manipulations",
          "displayName": "Graphics Manipulations",
          "riskCategory": "HIGH"
        }
      ],
      "completedFraudChecks": [
        "unusual-document-structure",
        "text-manipulations",
        "unusual-document-metadata",
        "editing-tool-usage",
        "graphics-manipulations"
      ]
    },
    {
      "createdDate": "2026-10-28T16:50:22.079",
      "agent": "Fraud Agent",
      "decision": "AGENT_SUSPECTED_FRAUD",
      "reason": "Suspicious documents",
      "fraudHints": [],
      "completedFraudChecks": []
    },
    {
      "createdDate": "2026-10-28T16:51:25.547",
      "agent": "Fraud Agent",
      "decision": "AGENT_CONFIRMED_FRAUD",
      "reason": "Example note from a human as to how this has been tampered.",
      "fraudHints": [],
      "completedFraudChecks": []
    }
  ],
  "profileAnalysis": [
    {
      "createdDate": "2026-10-28T14:59:53.548",
      "profileHints": [
        {
          "hintId": "STEEP_SPENDING_CURVE",
          "score": 0.005380542,
          "riskCategory": "LOW"
        }
      ]
    }
  ]
}
```

## Response fields

### History

Each entry in `history` is one fraud decision. A new entry is added each time a document finishes its tampering check
and each time an agent or third party system makes a fraud decision, so the list can be read as a timeline. Earlier
entries are never changed: a later decision, such as an agent confirming a case is not fraudulent, is added as a new
entry.

| Field                  | Description                                                                                                                                  |
|------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------|
| `createdDate`          | When the decision was made, in UTC                                                                                                           |
| `decision`             | The decision that was made. See [decisions](#decisions)                                                                                      |
| `reason`               | Why the decision was made. Automated checks use `System automated document tampering checks`. Agent decisions carry the reason they supplied |
| `documentId`           | The document that was checked. Only present for automated decisions; agent decisions apply to the whole case                                 |
| `agent`                | The name of the agent or system that made the decision, taken from their token. Only present for agent decisions                             |
| `fraudHints`           | The checks that raised a risk on the document. See [fraud hints](#fraud-hints). Always `[]` for agent decisions                              |
| `completedFraudChecks` | The ids of every tampering check group that ran on the document, whether it raised a risk or not. Always `[]` for agent decisions            |

#### Decisions

| Decision                                | Made by   | Description                                                                                                                                          |
|-----------------------------------------|-----------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `AUTOMATED_NOTHING_SUSPICIOUS_DETECTED` | Automated | The document's overall result was not suspicious, or the document was not sent for tampering checks                                                  |
| `AUTOMATED_SUSPECTED_FRAUD`             | Automated | The document's overall result was a warning. The document status becomes `SUSPECTED_FRAUD` and the case is referred to fraud for review              |
| `AUTOMATED_CONFIRMED_FRAUD`             | Automated | The document's overall result was high risk. The document status becomes `CONFIRMED_FRAUD`                                                           |
| `AGENT_SUSPECTED_FRAUD`                 | Agent     | An agent or third party system suspected the case is fraudulent, via `POST income/v2/incomeVerification/${incomeVerificationId}/suspectedFraud`      |
| `AGENT_CONFIRMED_FRAUD`                 | Agent     | An agent or third party system confirmed the case is fraudulent, via `POST income/v2/incomeVerification/${incomeVerificationId}/confirmFraud`        |
| `AGENT_CONFIRMED_NOT_FRAUD`             | Agent     | An agent or third party system confirmed the case is not fraudulent, via `POST income/v2/incomeVerification/${incomeVerificationId}/confirmNotFraud` |

An automated decision is based on the overall fraud result for the document, not on the highest risk among its
`fraudHints`. The tampering analysis scores every individual check it runs, weighting each by how strongly it indicates
fraud, and adds to the score when known combinations of checks fail together. The total score decides whether the
document is normal, suspected or confirmed fraud. A single tampering signal can be high risk on its own without being
strong enough evidence of fraud. That is why the first document in the example above has a `HIGH` hint but only an
`AUTOMATED_SUSPECTED_FRAUD` decision.

Whether an automated decision stops the case (and changes the income status, subStatus and document status) depends on your tenant config and on `disableFraudChecks`. See
[disable fraud checks](DisableFraudForTesting.md) and [what the statuses mean](GetIncomeVerificationState.md#what-do-the-different-statuses-mean).

#### Fraud hints

A fraud hint is a group of related tampering checks that raised a risk on a document. Each group is made up of several
individual checks on the PDF file, and the group takes the risk of its worst check.

| Field          | Description                                                                                                       |
|----------------|:------------------------------------------------------------------------------------------------------------------|
| `hintId`       | A stable id for the check group. Use it to match the hint to an entry in `completedFraudChecks`                   |
| `displayName`  | A human readable name for the check group                                                                         |
| `riskCategory` | The risk the group found: `MODERATE` for a warning, `HIGH` for high risk, or `UNDETERMINED` for an unknown result |

The check groups currently run are:

| Hint id                      | Display name               | Description                                                                                                                                        |
|------------------------------|----------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------|
| `text-manipulations`         | Text manipulations         | The way text is written into the file suggests it was edited, for example unexpected fonts, unusual text encoding or inconsistent text positioning |
| `graphics-manipulations`     | Graphics Manipulations     | Drawing instructions in the file suggest content was covered up or redrawn, for example shapes used to mask part of a page                         |
| `unusual-document-structure` | Unusual document structure | The internal structure of the file differs from what the issuer produces, for example signs of the file being updated after it was created         |
| `unusual-document-metadata`  | Unusual document metadata  | The file's metadata, such as its creator, producer, author or dates, differs from what the issuer produces                                         |
| `editing-tool-usage`         | Editing Tool Usage         | The file carries traces left by known PDF editing tools                                                                                            |
| `suspicious-bank`            | Suspicious bank            | The bank on the statement is on your tenant's custom list of suspicious banks. Bank statements only. No banks are added by default                 |

New check groups may be added, so treat `hintId` as an open set.

Check groups that found nothing suspicious are left out of `fraudHints`, but are still listed in `completedFraudChecks`.
An empty `fraudHints` list with a populated `completedFraudChecks` list means every check group ran and passed. If some
checks in a group could not complete, the group is reported as `MODERATE`.

An empty `completedFraudChecks` list on an automated decision means the document was not sent for tampering checks.
This happens for documents that are not bank statements or payslips, documents whose data could not be extracted, and
bank statements from banks that are not enabled for tampering checks.

### Profile analysis

Each entry in `profileAnalysis` is one run of the applicant profile analysis. The analysis runs on the bank statement
transactions, and runs again each time the transactions on the case change, such as when another bank statement is
added. The last entry is the most recent result. Cases with only payslips have no profile analysis.

| Field          | Description                                   |
|----------------|:----------------------------------------------|
| `createdDate`  | When the analysis ran, in UTC                 |
| `profileHints` | The profile checks that ran and their results |

| Field          | Description                                                                                                                              |
|----------------|:-----------------------------------------------------------------------------------------------------------------------------------------|
| `hintId`       | The profile check. Currently only `STEEP_SPENDING_CURVE`, a model that scores the applicant's spending pattern across the bank statement |
| `score`        | The score the model produced                                                                                                             |
| `riskCategory` | The risk the score falls into: `LOW`, `MODERATE`, `HIGH`, or `UNDETERMINED`                                                              |

`hintId` and `riskCategory` are extensible enums, so new values may be added. Treat unknown values as `UNDETERMINED`
rather than failing.

## Errors

The `401` and `403` responses that apply to every call are described in the
[security section](../../guides/security/CreatingJsonWebToken.md).

| HTTP Code | Error Type                      | Description                                                                                                           |
|-----------|---------------------------------|:----------------------------------------------------------------------------------------------------------------------|
| `404`     | `income-verification-not-found` | No case exists for the given `incomeVerificationId`                                                                   |
| `410`     | `entity-gone`                   | The case was deleted by the data retention policy. See [data retention](../../guides/data-retention/DataRetention.md) |

```json
{
  "error": {
    "errorType": "income-verification-not-found",
    "httpCode": 404,
    "traceId": "6abe06b37ed0754743e3feacdfa03bbc",
    "errors": [],
    "timestamp": "2026-10-01T07:07:31.686431993Z"
  }
}
```
