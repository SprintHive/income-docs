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
        "fullName": "Joe Soap",
        "grossIncome": 22000,
        "nettIncome": 18226.92,
        "idNumber": "9999999999",
        "bankAccountNumber": "99999999",
        "taxNumber": "9999999999",
        "timestamp": "2021-12-31",
        "totalDeductions": 3773.08,
        "totalEarnings": 22000,
        "uif": 148.72,
        "companyUifContribution": 148.72
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