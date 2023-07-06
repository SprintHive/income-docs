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
