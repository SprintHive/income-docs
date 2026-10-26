# Other Income

Other income is any income the applicant receives on top of their primary salary, for example a second job, rental
income or maintenance payments. It is declared when the case is created and is detected from the applicant's bank
statements alongside the primary income.

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)

## Declaring other income

Other income is declared in `otherIncome` when [creating the income verification](CreateIncomeVerificationRequest.md).
It is a list, with each entry in the same shape as `primaryIncome`.

Endpoint: ```income/v2/incomeVerification```  
Method: POST  
Body:
```json
{
  "config": {
    "minTransactionDaysRequired": 65,
    "bankStatementAndPayslipRequired": false,
    "maxAgeDays": {
      "bank-statement": 32
    }
  },
  "correlationId": "test-1",
  "primaryIncome": {
    "nettIncome": "20000",
    "grossIncome": "25000"
  },
  "otherIncome": [
    {
      "nettIncome": "4500",
      "payCycleInDays": 30
    },
    {
      "nettIncome": "1200",
      "payCycleInDays": 7
    }
  ]
}
```

| Field                          | Required | Description                                                                                                                                                                                      |
|--------------------------------|----------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `otherIncome`                  | No       | The list of other income streams. Leave it out or send `[]` if the applicant has none. There is no limit on its length                                                                           |
| `otherIncome[].nettIncome`     | Yes      | The nett amount the applicant receives per pay cycle. It must be a positive number                                                                                                               |
| `otherIncome[].payCycleInDays` | No       | The pay cycle that the `detectedNettIncome` is [scaled to when returned](#pay-cycle-and-the-detected-amount) (e.g. 7 for weekly, 14 for fortnightly, 30 for monthly). Must be a positive integer |
| `otherIncome[].grossIncome`    | No       | The gross amount. It is stored with the case but is not used to detect income                                                                                                                    |

> `payCycleInDays` defaults to 30

`nettIncome` is always required for an other income entry.
An invalid entry is rejected with a `400`, and the error names the index of the entry, e.g. `otherIncome[1].nettIncome`.
See the [create errors](CreateIncomeVerificationRequest.md#errors) for the full list.

### Pay cycle and the detected amount

`detectedNettIncome` is scaled to the declared `payCycleInDays` when reported back, so declare `nettIncome` per that cycle.
For an applicant paid R1000 every week:

| declared `payCycleInDays` | returned `detectedNettIncome` | returned `payCycle` |
|---------------------------|-------------------------------|---------------------| 
| `7`                       | `1000.00`                     | `7`                 |
| `14`                      | `2000.00`                     | `7`                 |
| `30`                      | `4285.71`                     | `7`                 |

`payCycle` in the result is the cycle found in the bank statement (here `7`), not the declared one.

## How other income is detected

* Other income is only detected from **bank statements**. Payslips are not used for other income.
* Primary income is detected first, and the transactions that make up the primary income are excluded.
* The entries in `otherIncome` are then matched **in the order they were declared**. Once a transaction has been matched
  to an entry it is not available to the entries after it, so list the larger or more regular income first.
* For each entry the system looks for a regular stream of credits (weekly, fortnightly or monthly) and picks the
  stream whose amount is closest to the declared `nettIncome`, scaled to the declared pay cycle.
* By default, credits of R300 or less are not considered income, so an other income paid in amounts of R300 or less will not be detected.
* When the `irregular` [income detector strategy](CreateIncomeVerificationRequest.md#incomedetectorstrategy) is used,
  other income is not detected and `otherIncome` is not returned in the result.

## Reading the result

The detected other income is returned by [Get Income Verification State](GetIncomeVerificationState.md) in
`result.analysisResult.otherIncome`. There is one item for each declared entry, in the same order as they were declared,
so `otherIncome[0]` in the result is the detected value for `otherIncome[0]` in the create request.

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}/state```  
Method: GET  
Response:
```json
{
  "status": "SUCCESS",
  "subStatus": "HIGH_INCOME_CONFIDENCE",
  "result": {
    "method": "AUTOMATED",
    "status": "HIGH_CONFIDENCE",
    "analysisResult": {
      "primaryIncome": {
        "declaredNettIncome": 20000,
        "detectedNettIncome": 19850.00,
        "confidence": 0.9984,
        "variance": -0.00756,
        ...
      },
      "otherIncome": [
        {
          "declaredNettIncome": 4500,
          "detectedNettIncome": 4400.00,
          "confidence": 0.9945,
          "variance": -0.02273,
          "payments": [
            {
              "date": "2021-05-03",
              "amount": 4300.00
            },
            {
              "date": "2021-06-01",
              "amount": 4500.00
            },
            {
              "date": "2021-07-01",
              "amount": 4400.00
            }
          ],
          "payDay": 1,
          "payCycle": 30,
          "docIds": [
            "d3107619-26d9-4093-a7e4-79fd9d4caeb2"
          ]
        },
        {
          "declaredNettIncome": 1200,
          "payments": [],
          "docIds": [
            "d3107619-26d9-4093-a7e4-79fd9d4caeb2"
          ]
        }
      ]
    }
  }
}
```

In the example above the first other income was found and the second was not.

| Field                               | Description                                                                                                                                 |
|-------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|
| `declaredNettIncome`                | The `nettIncome` declared for this entry                                                                                                    |
| `detectedNettIncome`                | The nett income detected for this entry, [scaled to the declared pay cycle](#pay-cycle-and-the-detected-amount). `null` if it was not found |
| `confidence`                        | A value between 0 and 1 showing how closely the detected income matches the declared income. `null` if not found                            |
| `variance`                          | `(detected - declared) / detected`. Negative when less was detected than declared. `null` if not found                                      |
| `payments`                          | The bank statement credits that make up this income. Empty if not found                                                                     |
| `payDay`                            | The day of the month this income is most often paid                                                                                         |
| `payCycle`                          | The pay cycle found in the bank statement in days (7, 14 or 30). This can differ from the declared pay cycle                                |
| `docIds`                            | The bank statements that were analysed                                                                                                      |

`payDayOfWeek`, `nettIncomeFromPayslip` and `payslipRelativeIncomeSource` are not populated for automatically detected other income.

Until the bank statements have been analysed, each item only contains `declaredNettIncome`.

### Confidence and passHigherThanDeclaredIncome

When `passHigherThanDeclaredIncome` is true (from tenant config or the create request config) and the detected other
income is higher than the declared amount, its confidence is 1.

## Effect on the case outcome

Other income does **not** affect the case `status`, `subStatus` or `result.status`. These are determined by the primary
income only. A case can be `SUCCESS` / `HIGH_INCOME_CONFIDENCE` while an other income entry was not found, and
[minIncome](MinIncome.md) is only compared to the primary income.

If other income matters to your decision, check `detectedNettIncome` and `confidence` for each entry in
`analysisResult.otherIncome` yourself.

## Transactions used

The [transactions endpoint](GetTransactionsUsed.md) marks the credits matched to any other income entry with
`"inOtherIncomeStream": true`. A transaction is never in both the primary income stream and an other income stream.

## Manual capture

When income is [captured manually](ManualCaptureIncome.md), the `otherIncome` sent in the manual capture **replaces** the
detected other income. Each entry's `declaredAmount` is returned as `declaredNettIncome`, and its confidence is 1.
If `otherIncome` is left out of the manual capture, `analysisResult.otherIncome` will be empty, so send every other
income you want to keep.

## Income confirmation

The other income used for the application can be sent back in `otherIncome` on the
[income confirmation](IncomeConfirmation.md). It is returned under `confirmation.otherIncome` in the state response.
