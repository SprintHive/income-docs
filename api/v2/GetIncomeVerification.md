# Get Income Verification

The following endpoint can be used to fetch an existing income verification case by its `incomeVerificationId`.
It returns the details the case was created with: the correlation id, the config, the declared income, the applicant details and
the created date.

This endpoint does not return the case status, documents or income result. Use the
[income verification state](GetIncomeVerificationState.md) endpoint for those.

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)
* Your token has the `incomeVerification:read` scope

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}```  
Method: GET  
Response:
```json
{
  "config": {
    "minTransactionDaysRequired": 65,
    "minPayslipsRequired": 1,
    "timeOutMinutes": 1440,
    "bankStatementAndPayslipRequired": false,
    "limitTransactionsByCutOffDate": true,
    "determineIncomeOnTimeOut": false,
    "maxAgeDays": {
      "bank-statement": 10000,
      "payslip": 10000
    },
    "incomeDetectorStrategy": "average",
    "documentTypeWhiteList": [
      "bank-statement",
      "payslip"
    ],
    "disableFraudChecks": true
  },
  "incomeVerificationId": "49f9290c-d393-4af4-8072-0400303f64c8",
  "correlationId": "test-1",
  "createdDate": "2026-09-30T14:34:37.746",
  "primaryIncome": {
    "grossIncome": 25000.00,
    "nettIncome": 20000.00,
    "payCycleInDays": 30
  },
  "otherIncome": []
}
```

## Response fields

| Field                  | Description                                                                                                                  |
|------------------------|:-----------------------------------------------------------------------------------------------------------------------------|
| `incomeVerificationId` | The id of the case                                                                                                           |
| `correlationId`        | Your system's identifier for the case, as supplied on creation                                                               |
| `createdDate`          | When the case was created                                                                                                    |
| `config`               | The config the case was created with. See [configuration options](CreateIncomeVerificationRequest.md#configuration-options) |
| `primaryIncome`        | The declared primary income: `nettIncome`, `grossIncome` and `payCycleInDays`                                                |
| `otherIncome`          | Any other declared income, in the same shape as `primaryIncome`                                                              |
| `applicantDetails`     | The applicant details supplied on creation. Left out when none were supplied                                                 |
| `invMetadata`          | Optional metadata linked to the case, such as the related product                                                           |

Fields that were not supplied on creation may be `null` or left out of the response.

## Errors

The `401` and `403` responses that apply to every call are described in the
[security section](../../guides/security/CreatingJsonWebToken.md).

| HTTP Code | Error Type                        | Description                                                                                                  |
|-----------|-----------------------------------|:-------------------------------------------------------------------------------------------------------------|
| `404`     | `income-verification-not-found`   | No case exists for the given `incomeVerificationId`                                                          |
| `410`     | `entity-gone`                     | The case was deleted by the data retention policy. See [data retention](../../guides/data-retention/DataRetention.md) |

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
