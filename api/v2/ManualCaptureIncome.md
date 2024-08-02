# Manually capture income

This endpoint can be used to capture the income result from manually viewing the income documents.

## Prerequisites
* You have read and understand the security section [click here for more details](../../guides/security/CreatingJsonWebToken.md)

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}/manualCapture/manualIncome```  
Method: POST  
Body:
```json
{
  "primaryIncome": {
    "payments": [
      {
        "date": "2021-06-29",
        "amount": 4238.22
      }
    ],
    "declaredAmount": 4500,
    "payDay": "29",
    "payCycle": "30"
  },
  "otherIncome": [
    {
      "payments": [
        {
          "date": "2021-06-17",
          "amount": 950.0
        }
      ],
      "declaredAmount": 1000,
      "payDay": "17",
      "payCycle": "30"
    }
  ],
  "accountHolderName": "Mr Raoul McClureBee",
  "bankName": "Nedbank",
  "bankAccountNumber": "1180234567"
}
```

Response:
No body is returned for this response

