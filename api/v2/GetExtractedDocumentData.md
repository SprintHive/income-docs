# Getting the extracted data for a document

The following endpoint can be used to download the data that was extracted from the document.

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)

To get a list of the documents linked to an income verification request [see the get state API](./GetIncomeVerificationState.md)

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}/document/${documentId}```
Method: GET  

An example of the response when the document is a payslip: 
```json
{
    "type": "PAYSLIP",
    "status": "DATA_EXTRACTION_COMPLETED",
    "hash": "eae67d0fd9570c2898a5db14efc4d681a8a6e2a6",
    "received": "2024-01-12T06:52:44.232",
    "payslip": {
        "dateEngaged": "2016-01-01",
        "employer": "ACME Pty (Ltd)",
        "employeeAddress": "1 Blank Street Nowhere 0000",
        "fullName": "Joe Soap",
        "grossIncome": 22000,
        "nettIncome": 18226.92,
        "idNumber": "9999999999",
        "bankAccountNumber": "99999999",
        "branchCode": "99999",
        "bank": "ACME bank",    
        "taxNumber": "9999999999",
        "timestamp": "2021-12-31",
        "totalDeductions": 3773.08,
        "totalEarnings": 22000,
        "uif": 148.72,
        "companyUifContribution": 148.72,
        "jobTitle": "Clerk",
        "earningItems": [
          {
            "description": "Cash Salary",
            "type": "salary",
            "amount": 14294.16
          }
        ],
        "deductionItems": [
          {
            "description": "Total Tax",
            "type": "other-deductions",
            "amount": 1424.62
          },
          {
            "description": "Gap Cover",
            "type": "other-deductions",
            "amount": 144.00
          },
          {
            "description": "Payroll Giving",
            "type": "other-deductions",
            "amount": 30.00
          },
          {
            "description": "UIF Deduction",
            "type": "uif",
            "amount": 148.72
          }
      ]
    }
}
```

An example of the response when the document is a bank statement
```json
{
    "type": "BANK_STATEMENT",
    "status": "DATA_EXTRACTION_COMPLETED",
    "hash": "20551bbd54e35dc0621e764714bfa590d24b6083",
    "received": "2024-01-12T06:51:32.276",
    "bankStatement": {
        "transactions": [
            {
                "id": "28042230-f2f5-400f-8589-480b7d315694",
                "date": "2021-10-31",
                "description": "Payment Received: Salaries Wages",
                "balance": 19499.65,
                "transactionValue": 19476.92,
                "tags": []
            },
          ...
        ],
        "creationDate": "2024-01-12T06:51:33.348899",
        "name": "Joe Soap",
        "address": "ACME Pty (Ltd)\n23 Brick Lane\nACME Town - 9999",
        "bankAccount": {
            "accountNumber": "9999999999",
            "bank": "Capitec Bank"
        },
        "balanceBroughtForward": 22.73,
        "statementStartDate": "2021-10-28",
        "statementEndDate": "2022-01-26"
    }
}
```

An example of the response when the document is a bank statement with additional transaction enrichment
```json
{
    "type": "BANK_STATEMENT",
    "status": "DATA_EXTRACTION_COMPLETED",
    "hash": "20551bbd54e35dc0621e764714bfa590d24b6084",
    "received": "2024-01-12T06:51:32.276",
    "bankStatement": {
        "transactions": [
            {
                "id": "62042230-f2f5-400f-8589-480b7d315662",
                "date": "2022-03-20",
                "description": "FEE-POS DECLINED INSUFF FUNDS Checkers Long beach 5709",
                "balance": 12.45,
                "transactionValue": -7.90,
                "failureIndicator": true,
                "thirdParty": {
                  "name": "Checkers",
                  "subDivision": "Checkers"
                },
                "category": "Groceries",
                "transactionType": "Fee Deduction",
                "tags": ["Groceries"]
            },
          ...
        ],
        "creationDate": "2024-01-12T06:51:33.348899",
        "name": "Jane Soap",
        "address": "ACME Pty (Ltd)\n23 Brick Lane\nACME Town - 9999",
        "bankAccount": {
            "accountNumber": "9999999999",
            "bank": "Absa Bank"
        },
        "balanceBroughtForward": 22.73,
        "statementStartDate": "2022-02-25",
        "statementEndDate": "2022-03-26"
    }
}
```

### failureIndicator
Failed Debit Order / Failed Transaction Detection: 
Transactions that indicate failed transaction attempts are marked. This will include reversed transactions, insufficient funds fees, failed card transactions, failed debit orders etc.

### category
Expense Classification: 
Bank statement transactions are classified into expense categories so that affordability and risk checks can be performed.

### thirdParty
3rd Party Identification: 
Where possible the other party involved in the transaction is identified. This includes large retailers, financial services, online stores, and many more.

### transactionType
Transaction type detection: 
The transaction type is marked on the transaction where possible e.g. card transaction, debit order, online transfer, stop order etc.
