# Creating an Income Verification Request with income from bank statement and payslip required

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)


# When bankStatementAndPayslipRequired is true

When bankStatementAndPayslipRequired is true when creating an income verification (Endpoint: `income/v2/incomeVerification`), we
will get a bank statement income result and a payslip income result once valid documents of each type have been added 
to the case. These income results will be compared to the declared nett income from the customer. Both the result from 
the bank statements and payslips must be high confidence for the case to be status: `SUCCESSFUL`, 
subStatus: `HIGH_CONFIDENCE`.

If either result is low confidence, the case subStatus will also be `LOW_CONFIDENCE`.

# When bankStatementAndPayslipRequired and tenant config bankStatementRelativeToPayslipEnabled are true

We will still get a bank statement income result and a payslip income result once valid documents have been
added to the case, but with this config, we can also have income detected from the bank statement relative to the payslip
income instead of the declaration.

## Nullable declared income
The **declared nett income** when creating an income verification is also now **optional**. Whereas otherwise, 
primaryIncome.nettIncome would be required.
Endpoint: ```income/v2/incomeVerification```  
Method: POST  
Example of valid Body with no primaryIncome:
```json
{
  "config": {
    "minTransactionDaysRequired": 65,
    "minPayslipsDaysRequired": 1,
    "timeOutMinutes": 1440,
    "bankStatementAndPayslipRequired": true,
    "incomeDetectorStrategy": "average",
    "maxAgeDays": {
      "bank-statement": 32,
      "payslip": 32
    }
  },
  "correlationId": "test-1"
}
```
Example of valid Body with no nettIncome in primaryIncome:
```json
{
  "config": {
    "minTransactionDaysRequired": 5,
    "minPayslipsDaysRequired": 1,
    "timeOutMinutes": 15,
    "bankStatementAndPayslipRequired": true,
    "incomeDetectorStrategy": "average",
    "maxAgeDays": {
      "bank-statement": 32,
      "payslip": 32
    }
  },
  "correlationId": "test-2",
  "primaryIncome": {
    "payCycleInDays": 30
  }
}
```

## Bank statement income
If we have declared income, we will detect income from the bank statement relative to the declaration. The confidence
will also be calculated relative to the declaration.

When we have a high confidence payslip, we will detect income from the bank statement relative to the payslip's income.
The confidence will also be calculated relative to the payslip's income.

## Result
If we have both bank statement income relative to declared and relative to payslip. We will use the one with the
higher confidence as the bank statement result. If we only have one of these results, that will be used. Then the bank
statement and payslip results are considered as usual (described above).

## Relative income source
To see what the results used were relative to, see `bankStatementRelativeIncomeSource` and `payslipRelativeIncomeSource`
in the state result. Endpoint: income/v2/incomeVerification/${incomeVerificationId}/state
They can be:
- `DECLARED` Compared to the declared nett income.
- `PAYSLIP` When the bank statement income was calculated relative to the payslip.
- `NONE` When the declaration was null, and the payslip was not compared to anything.
