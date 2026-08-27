# Fetching document chase information for an income verification 

The following endpoint can be used to fetch information about what income documents have been attached to an income 
verification, and what outstanding documents are still required to validate the business rules.
This information could be used to communicate to customers about what documents they need to send with more specificity. 

Contact SprintHive if you would like to enable this api.

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}/documentChase```  
Method: GET  

Below is an example response for an income verification where 3 months bank statements or 3 payslips are required.
The bank statement for July has been supplied but two more months are required. The May and July payslips have been 
supplied, but at least one more payslip is required and there cannot be a month missing in the middle (they must be 
contiguous). 
The result is calculated as though the api was called on 2025-07-30.

Response:
```json
{
  "documentRequirementsMet": false,
  "bankStatementAndPayslipRequired": false,
  "bankStatementChase": {
    "bankStatementRequirementsMet": false,
    "lastDocumentReceivedDate": "2025-07-30",
    "minTransactionDaysRequired": 65,
    "minMonthsRequired": 3,
    "transactionDaysPresent": 26,
    "transactionDateRangesPresent": [
      {
        "startDate": "2025-06-29",
        "endDate": "2025-07-25"
      }
    ],
    "noOfMonthsPresent": 1,
    "bankStatementMonthsPresent": [
      {
        "year": 2025,
        "month": 7
      }
    ],
    "isContiguous": true,
    "transactionDaysOutstanding": 40,
    "noOfMonthsOutstanding": 2,
    "acceptableDateRangesForAdditionalBankStatements": [
      {
        "startDate": "2025-04-26",
        "endDate": "2025-06-29"
      },
      {
        "startDate": "2025-07-25",
        "endDate": "2025-07-30"
      }
    ],
    "bankStatementMonthsOutstanding": [
      {
        "year": 2025,
        "month": 5
      },
      {
        "year": 2025,
        "month": 6
      }
    ]
  },
  "payslipChase": {
    "payslipRequirementsMet": false,
    "lastDocumentReceivedDate": "2025-07-29",
    "minPayslipsRequired": 3,
    "noOfPayslipsPresent": 2,
    "payslipDatesPresent": [
      "2025-05-24",
      "2025-07-25"
    ],
    "payslipMonthsPresent": [
      {
        "year": 2025,
        "month": 5
      },
      {
        "year": 2025,
        "month": 7
      }
    ],
    "isContiguous": false,
    "noOfPayslipsOutstanding": 1,
    "acceptableDateRangeForAdditionalPayslips": {
      "startDate": "2025-04-29",
      "endDate": "2025-07-30"
    },
    "payslipMonthsOutstanding": [
      {
        "year": 2025,
        "month": 6
      }
    ]
  }
}
```
# Response Description
Firstly `documentRequirementsMet` shows whether the documents required for income verification have been attached to
the case. If this is true then no further chasing for documents should be required.
`bankStatementAndPayslipRequired` is a config set when the income verification is created. If true, the requirements
for bank statements and payslips must be met. If false, the case can be completed using either of the document types.

Fields that have no value are omitted from the JSON response rather than returned as `null`. The **Presence** column in
the tables below indicates when a field may be absent.

Two composite types appear in the tables below:

- date range object — `{ "startDate": "2025-04-26", "endDate": "2025-06-29" }`
- year/month object — `{ "year": 2025, "month": 7 }`

## Bank statement result

`bankStatementChase` is only present when bank statements are enabled for the case. This is controlled by
`documentTypeWhiteList` on the [create income verification request](CreateIncomeVerificationRequest.md), which falls
back to the service level config when not supplied. If the white list is set and does not include `bank-statement`, the
whole `bankStatementChase` object is absent from the response.

| Field                        | Description                                                                                                                                                                                                                                                                                                          | Type                             | Presence                                                       |
|------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------|:---------------------------------------------------------------|
| bankStatementRequirementsMet | This is true if sufficient bank statements are present, false if the bank statement requirements have not been met.                                                                                                                                                                                                  | `boolean`                        | Always present                                                 |
| lastDocumentReceivedDate     | This is the last date when a valid bank statement was uploaded to the income verification. This date will not be updated for a bank statement that is entirely too old or does not match the applicant.                                                                                                              | `string`, a date in `yyyy-MM-dd` | Absent until the first valid bank statement has been processed |
| minTransactionDaysRequired   | This is the minimum number of days required between the first transaction to the last transaction from the bank statements. There need not be transactions on every day in between.                                                                                                                                  | `integer`                        | Always present                                                 |
| minMonthsRequired            | If there are additional transactions present beyond the `minTransactionDaysRequired` requirement, up to `minMonthsRequired` worth of data will be used for determining income. `minMonthsRequired` is rounded up from the `minTransactionDaysRequired`. Eg 15 days to 1 month, 65 days to 3 months, 165 to 6 months. | `integer`                        | Always present                                                 |

**What is present**

| Field                        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Type                        | Presence                                                           |
|------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------|:-------------------------------------------------------------------|
| transactionDaysPresent       | The number of days between the first and last transaction present that are recent enough, excluding missing days if the bank statements are not contiguous.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `integer`                   | Always present; `0` when no usable transactions                    |
| transactionDateRangesPresent | The valid date ranges of bank statement transactions that are present.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | array of date range objects | Always present; `[{}]` when no usable transactions are present yet |
| bankStatementMonthsPresent   | This list of calendar months is derived from the `transactionDateRangesPresent`. For each date range, the number of months it represents is calculated by taking the number of days it covers and dividing by 30.5 (the approximate average number of days in a month), then rounding to the nearest integer "X" (with a minimum of 1). The calendar months touched by that date range are then sorted from most days covered to least, and the top "X" months are counted as "present". Each present date range is evaluated independently. If the bank statements are not contiguous, the missing calendar months (see `bankStatementMonthsOutstanding` below) are first excluded from the candidate months before this selection is made, so a month claimed by the gap cannot also be counted as present. See `transactionDateRangesPresent` for a more detailed view of the exact date ranges present. | array of year/month objects | Always present; `[]` when no months are present                    |
| noOfMonthsPresent            | This is the number of calendar months deemed to be present in `bankStatementMonthsPresent`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `integer`                   | Always present; `0` when no months are present                     |
| isContiguous                 | This will be true if there are no months missing between bank statements. If `isContiguous` is false, then there is at least a month missing between two other bank statements uploaded.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | `boolean`                   | Absent until the statements have been checked for contiguity       |

**What is outstanding**

| Field                                           | Description                                                                                                                                                                       | Type                        | Presence                                                                                    |
|-------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------|:--------------------------------------------------------------------------------------------|
| transactionDaysOutstanding                      | The number of additional days the transaction must cover to pass validation.                                                                                                      | `integer`                   | Always present; `0` when nothing is outstanding                                             |
| noOfMonthsOutstanding                           | The number of additional months needed to pass validation.                                                                                                                        | `integer`                   | Always present; `0` when nothing is outstanding                                             |
| acceptableDateRangesForAdditionalBankStatements | The list of valid date ranges for additional bank statements not already covered by present bank statements.                                                                      | array of date range objects | Always present with at least one entry; entries always carry both `startDate` and `endDate` |
| bankStatementMonthsOutstanding                  | The list of calendar months of bank statements that should be uploaded to pass validation. For non-contiguous bank statements, the missing calendar months are prioritised first. | array of year/month objects | Always present; `[]` when nothing is outstanding                                            |

## Payslip result

`payslipChase` is only present when payslips are enabled for the case. This is controlled by `documentTypeWhiteList` on
the [create income verification request](CreateIncomeVerificationRequest.md), which falls back to the service level
config when not supplied. If the white list is set and does not include `payslip`, the whole `payslipChase` object is
absent from the response.

| Field                    | Description                                                                                                                                                                      | Type                             | Presence                                                |
|--------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------|:--------------------------------------------------------|
| payslipRequirementsMet   | This is true if sufficient payslips are present, false if the payslip requirements have not been met.                                                                            | `boolean`                        | Always present                                          |
| lastDocumentReceivedDate | This is the last date when a valid payslip was uploaded to the income verification. This date will not be updated for a payslip that is too old or does not match the applicant. | `string`, a date in `yyyy-MM-dd` | Absent until the first valid payslip has been processed |
| minPayslipsRequired      | This is the minimum number of payslips required to pass validation.                                                                                                              | `integer`                        | Always present                                          |

**What is present**

| Field                | Description                                                                                                                                                                                                                     | Type                                           | Presence                                                                                                           |
|----------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------|:-------------------------------------------------------------------------------------------------------------------|
| noOfPayslipsPresent  | The number of valid payslips that are attached to this income verification.                                                                                                                                                     | `integer`                                      | Always present; `0` when no payslips are present                                                                   |
| payslipDatesPresent  | The document dates of the payslips present.                                                                                                                                                                                     | array of `string`, each a date in `yyyy-MM-dd` | Absent until the first payslip has been processed; `[]` when every payslip present is too old or otherwise invalid |
| payslipMonthsPresent | The list of calendar months that have a payslip present.                                                                                                                                                                        | array of year/month objects                    | Always present; `[]` when no payslips are present                                                                  |
| isContiguous         | This will be true if there are no large gaps exceeding the pay cycle (monthly, fortnightly or weekly) between payslips. If `isContiguous` is false, then there is a gap of missing payslips between the other payslips present. | `boolean`                                      | Absent until the payslips have been checked for contiguity                                                         |

**What is outstanding**

| Field                                    | Description                                                                                                                                                                                                                                         | Type                        | Presence                                                      |
|------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------|:--------------------------------------------------------------|
| noOfPayslipsOutstanding                  | The number of additional payslips required to meet the `minPayslipsRequired`. Note that `noOfPayslipsOutstanding` could be 1 if the payslips are not contiguous (there is a large gap between payslips) even if `minPayslipsRequired` has been met. | `integer`                   | Always present; `0` when nothing is outstanding               |
| acceptableDateRangeForAdditionalPayslips | The valid date range for payslips.                                                                                                                                                                                                                  | date range object           | Always present; always carries both `startDate` and `endDate` |
| payslipMonthsOutstanding                 | The list of calendar months of payslips that should be uploaded to pass validation.                                                                                                                                                                 | array of year/month objects | Always present; `[]` when nothing is outstanding              |

## A use case example

- An income verification for a customer is [created](CreateIncomeVerificationRequest.md) requiring the most recent 3 months bank statement or 3 months payslips.

- The customer uploads some documents, and you receive a [document processed or status changed notification](../../guides/notifications/Notifications.md).

- You check the [state of the income verification](GetIncomeVerificationState.md) and the status is IN_PROGRESS with the subStatus being one of
WAITING_FOR_DOCUMENTS, PROBLEMS_WITH_DOCUMENTS or DATA_EXTRACTION_FAILED. This means that the document requirements
necessary for the income verification have not been met yet.

- You get the documentChase results for this income verification using the api outlined above and use the information to 
craft a communication to the customer requesting additional documents.

Say you get the example response above.
`documentRequirementsMet`, `bankStatementRequirementsMet` and `payslipRequirementsMet` are all false.

### Example messages:
"Dear customer, we have received 1 {noOfMonthsPresent} bank statement(s) from July {bankStatementMonthsPresent}. In order to satisfy our 
requirement of 3 {minMonthsRequired} months of recent consecutive bank statements please upload 2 {noOfMonthsOutstanding} additional statements 
for May and June {bankStatementMonthsOutstanding}."

"Dear customer, we have received 2 {noOfPayslipsPresent} payslips from May and July {payslipMonthsPresent}. We require 3 {minPayslipsRequired} recent consecutive 
payslips. There is a gap between your uploaded payslips {isContiguous:false}. Please upload 1 {noOfPayslipsOutstanding}
additional payslip(s) for June {payslipMonthsOutstanding}."

- Some version of a message is sent to the customer requesting particular months of payslips and bank statements.
- The customer sends additional documents which pass validation. Income is detected and successfully automated.  🥳
