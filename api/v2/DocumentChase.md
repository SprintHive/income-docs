# Fetching document chase information for an income verification 

The following endpoint can be used to fetch information about what income documents have been attached to an income 
verification, and what outstanding documents are still required to validate the business rules.
This information could be used to communicate to customers about what documents they need to send with more specificity. 

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}/documentChase```  
Method: GET  
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
    "bankStatementMonthsPresent": [
      {
        "year": 2025,
        "month": 7
      }
    ],
    "isContiguous": true,
    "noOfTransactionDaysOutstanding": 40,
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
        "month": 4
      },
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
        "month": 4
      },
      {
        "year": 2025,
        "month": 6
      }
    ]
  }
}
```
# Response Description
Firstly `documentRequirementsMet` show whether the documents required for income verification have been attached to 
the case. If this is true then no further chasing for documents should be required.
`bankStatementAndPayslipRequired` is a config set when the income verification is created. If true, the requirements 
for bank statements and payslips must be met. If false, the case can be completed using either of the document types.

## Bank statement result

| Field                                           | Description                                                                                                                                                                                                                                                                                                          |
|-------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| bankStatementRequirementsMet                    | This is true if sufficient bank statements are present, false if the bank statement requirements have not been met.                                                                                                                                                                                                  |
| lastDocumentReceivedDate                        | This is the last date when a valid bank statement was uploaded to the income verification. This date will not be updated for a bank statement that is entirely too old or does not match the applicant.                                                                                                              |
| minTransactionDaysRequired                      | This is the minimum number of days required between the first transaction to the last transaction from the bank statements. There need not be transactions on every day inbetween.                                                                                                                                   |
| minMonthsRequired                               | If there are additional transactions present beyond the `minTransactionDaysRequired` requirement, up to `minMonthsRequired` worth of data will be used for determining income. `minMonthsRequired` is rounded up from the `minTransactionDaysRequired`. Eg 15 days to 1 month, 65 days to 3 months, 165 to 6 months. |

**What is present**

| Field                                           | Description                                                                                                                                                                                                                                                                                                      |
|-------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| transactionDaysPresent                          | The number of days between the first and last transaction present that are recent enough.                                                                                                                                                                                                                        |
| transactionDateRangesPresent                    | The valid date ranges of bank statement transactions that are present.                                                                                                                                                                                                                                           |   
| bankStatementMonthsPresent                      | This list of calendar months is derived from the `transactionDateRangesPresent`. If at least 15 days are covered in a calendar month, that month will be counted as "present". If no calendar months have 15 days of transactions then the single month with the most transactions will be counted as "present". |
| isContiguous                                    | This will be true if there are no large gaps between bank statements. If `isContiguous` is false, then there is at least a month missing between two other bank statements uploaded.                                                                                                                             |

**What is outstanding**

| Field                                           | Description                                                                                                                                                                                                                          |
|-------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| noOfTransactionDaysOutstanding                  | The number of additional days the transaction must cover to pass validation. Note that `noOfTransactionDaysOutstanding` could be 0 while the bank statement requirements are not met yet, if the bank statements are not contiguous. |
| acceptableDateRangesForAdditionalBankStatements | The list of valid date ranges for additional bank statements not already covered by present bank statements.                                                                                                                         |
| bankStatementMonthsOutstanding                  | The list of calendar months within the valid date range that does not have sufficient bank statement data yet (do not have `bankStatementMonthsPresent`).                                                                            |

## Payslip result

| Field                                 | Description                                                                                                                                                                      |
|---------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| payslipRequirementsMet                | This is true if sufficient payslip are present, false if the payslip requirements have not been met.                                                                             |
| lastDocumentReceivedDate              | This is the last date when a valid payslip was uploaded to the income verification. This date will not be updated for a payslip that is too old or does not match the applicant. |
| minPayslipsRequired                   | This is the minimum number of payslips required to pass validation.                                                                                                              |

**What is present**

| Field                              | Description                                                                                                                                                                                                                     |
|------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| noOfPayslipsPresent                | The number of valid payslips that are attached to this income verification.                                                                                                                                                     |
| payslipDatesPresent                | The document dates of the payslips present.                                                                                                                                                                                     |   
| payslipMonthsPresent               | This list of calendar months that have a payslip present.                                                                                                                                                                       |
| isContiguous                       | This will be true if there are no large gaps exceeding the pay cycle (monthly, fortnightly or weekly) between payslips. If `isContiguous` is false, then there is a gap of missing payslips between the other payslips present. |

**What is outstanding**

| Field                                    | Description                                                                                                                                                                                                      |
|------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| noOfPayslipsOutstanding                  | The number of additional payslips required to meet the `minPayslipsRequired`. Note that `noOfPayslipsOutstanding` could be 0 while the payslip requirements are not met yet, if the payslips are not contiguous. |
| acceptableDateRangeForAdditionalPayslips | The valid date range for payslips.                                                                                                                                                                               |
| payslipMonthsOutstanding                 | The list of calendar months within the valid date range that do not have a payslip.                                                                                                                              |

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
"Dear customer, we have received your July {bankStatementMonthsPresent} bank statement(s). In order to satisfy our 
requirement of 3 months of recent consecutive bank statements {minMonthsRequired} please upload additional statements 
from June, May and/or April {bankStatementMonthsOutstanding}."

"Dear customer, we have received your May and July {payslipMonthsPresent} payslip(s). We require 3 recent consecutive 
payslips {minPayslipsRequired}. There is a gap between your uploaded payslips {isContiguous:false}. Please upload 
additional payslips from June, and/or April {payslipMonthsOutstanding}."

- Some version of a message is sent to the customer requesting particular months of payslips and bank statements.
- The customer sends additional documents which pass validation. Income is detected and successfully automated.  🥳
