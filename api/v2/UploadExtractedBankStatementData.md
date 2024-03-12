## Uploading extracted bank statement data

The following endpoint can be used to upload extracted bank statement data for an income verification request.

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)

Endpoint: ```income/v2/incomeVerification/{incomeVerificationId}/extractedBankStatements```  
Method: POST  
Body:
```json
{
  "extractedBankStatements": [
    {
      "transactions": [
        {
          "id": "9892eb34-4050-400d-b3ce-17d7d4851380",
          "date": "2024-03-31",
          "description": "Payment Received: Salaries Wages 2017/03/31 Transfer 01234567",
          "balance": 19499.65,
          "transactionValue": 19476.92,
          "tags": []
        },
        
        ...

        {
          "id": "b9233039-3f99-499f-acf9-213be3cd449d",
          "date": "2024-06-26",
          "description": "USSD Prepaid Purchase CELLC",
          "balance": 25.91,
          "transactionValue": -12.00,
          "tags": []
        }
      ],
      "name": "John Doe",
      "address": "ACME Pty (Ltd)\n23 Brick Lane\nACME Town - 9999",
      "bankAccount": {
        "accountNumber": "bankAccountNumber123",
        "bank": "Capitec Bank"
      },
      "balanceBroughtForward": 22.73,
      "statementStartDate": "2024-03-28",
      "statementEndDate": "2024-06-26",
      "statementDate": "2024-03-28"
    }
  ]
}
```

Response:
- HTTP status code: 200

- HTTP status code: 400

## Required fields
- extractedBankStatements
  - transactions
    - date
    - transactionValue

Every transaction must have a date and transaction value.

## Field Validation

If statementStartDate and statementEndDate both exist, then the statementStartDate must be before the statementEndDate.

