# Disable Fraud Checks

You can **disable fraud checks** at the case level in testing environments by setting the `disableFraudChecks` parameter to `true` within the `config` object during a create call.

```json
{
  "config": {
    "minTransactionDaysRequired": 5,
    "minPayslipsRequired": 1,
    "timeOutMinutes": 15,
    "bankStatementAndPayslipRequired": false,
    "limitTransactionsByCutOffDate": true,
    "maxAgeDays": {
      "bank-statement": 1000,
      "payslip": 1000
    },
    "disableFraudChecks": true
  },
  "correlationId": "xxxxxx-xxxxxx-xxxxx-xxxxx",
  "declaration": {
    "grossIncome": 6000,
    "nettIncome": 5000,
    "payCycleInDays": 30
  },
  "otherIncome": []
}
```

When `disableFraudChecks` is `true`, documents still undergo fraud analysis, and their status could be **`confirmed fraud`**. However, Income won't halt the case immediately based on this status. The fraudulent document will still proceed through extraction and further processing, allowing the case to be completed and yield a final result.
When `disableFraudChecks` is `false` or it is left unset income will behave as normal, so there is no need to change existing calls in prod environments.

---

This functionality is controlled by a **tenant configuration** and is strictly limited to **testing environments**. Any attempt to disable fraud checks in a production environment will result in an `invalid-input 400` error:

```json
{
  "error": {
    "errorType": "invalid-input",
    "httpCode": 400,
    "traceId": "xxxxxx-xxxxxx-xxxxx-xxxxx",
    "errors": [
      {
        "field": "disableFraudChecks",
        "errorCode": "fraud-checks-cannot-be-disable"
      }
    ],
    "timestamp": "2020-01-01T00:00:00.0000000Z"
  }
}
```