## Sending the confirmed income

The following endpoint can be used to post the income that was used for the application to SprintHive.

Endpoint: ```income/v2/incomeVerification/${incomeVerificationId}/confirmation```  
Method: POST  
Body:
```json
{
  "primaryIncome": {
    "nettIncome": "20000",
    "grossIncome": "25000"
  },
  "otherIncome": [
    {
      "nettIncome": "4300",
      "grossIncome": "5000"
    }
  ],
  "applicationResult": "ACCEPTED",
    "documents": [
        {
            "documentId": "bc4d9546-fba6-4a16-b408-7989274a0996", 
            "documentHash": "d8dd6a0baeeb6eba9674a96e33c05586ce95e678"
        }
    ]
}
```

The confirmation is returned in the [Get Income Verification State](api/v2/GetIncomeVerificationState.md) endpoint. 

```json
{
    "status": "SUCCESS",
    "subStatus": "HIGH_INCOME_CONFIDENCE",
    "confirmation": {
        "primaryIncome": {
            "nettIncome": 4300.00
        },
        "otherIncome": [
            {
                "nettIncome": 4300.00,
                "grossIncome": 5000.00
            }
        ],
        "agentId": "an-agent-id",
        "applicationResult": "ACCEPTED",
        "documents": [
            {
                "documentId": "91eb98ee-74f8-4cef-9896-c1a3599012bd",
                "documentHash": "36420742ea5a848249bac915e7ba3c951cdda395"
            }
        ]
    }
}
```