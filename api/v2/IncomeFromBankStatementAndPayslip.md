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
The **declared nett income** when creating an income verification is also now **optional**, since the bank statement 
income can be compared to the payslip instead of the declaration. Whereas otherwise, primaryIncome.nettIncome would be 
a required field.

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
in the state result. 
They can be:
- `DECLARED` Compared to the declared nett income.
- `PAYSLIP` When the bank statement income was calculated relative to the payslip.
- `NONE` When the declaration was null, and the payslip was not compared to anything.

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}/state```  
Method: GET  
Example Response where the declared income was null and the bank statement was compared to the payslip:
```json
{
	"status": "SUCCESS",
	"subStatus": "HIGH_INCOME_CONFIDENCE",
	"documents": [
		{
			"documentId": "0bc0be9b-4ffc-4044-ac21-e42178593f82",
			"documentHash": "d03e4c0a47f73beccb66af894c4c94ec8c9c62ca",
			"type": "BANK_STATEMENT",
			"received": "2024-11-29T13:04:02.268",
			"status": "DATA_EXTRACTION_COMPLETED",
			"usedToDetectIncome": false,
			"sizeBytes": "92832",
			"textExtractionType": "MACHINE_READABLE",
			"contentType": "application/pdf"
		},
		{
			"documentId": "66de9886-c498-4bdd-ada1-3278a4b415b1",
			"documentHash": "47ee65e9561d5c316305a2fff73dc44a9b950001",
			"type": "PAYSLIP",
			"received": "2024-11-29T13:04:02.453",
			"status": "DATA_EXTRACTION_COMPLETED",
			"usedToDetectIncome": true,
			"sizeBytes": "58897",
			"textExtractionType": "MACHINE_READABLE",
			"contentType": "application/pdf"
		}
	],
	"result": {
		"method": "AUTOMATED",
		"status": "HIGH_CONFIDENCE",
		"bankName": "Absa Bank",
		"bankAccountNumber": "123456789",
		"analysisResult": {
			"primaryIncome": {
				"detectedNettIncome": 37242.91,
				"confidence": 1.0,
				"variance": 0.0,
				"payments": [
                  {
                    "date": "2024-09-24",
                    "amount": 37210.20
                  },
                  {
                    "date": "2024-10-25",
                    "amount": 37249.50
                  },
                  {
                    "date": "2024-11-25",
                    "amount": 37269.03
                  }
				],
				"payDay": 25,
				"payCycle": 30,
				"docIds": [
                  "66de9886-c498-4bdd-ada1-3278a4b415b1",
                  "0bc0be9b-4ffc-4044-ac21-e42178593f82"
				],
				"nettIncomeFromPayslip": 37269.03,
				"bankStatementRelativeIncomeSource": "PAYSLIP",
				"payslipRelativeIncomeSource": "NONE"
			},
			"otherIncome": []
		}
	}
}
```
