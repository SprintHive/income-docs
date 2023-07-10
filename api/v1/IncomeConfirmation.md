## Sending the confirmed income

The following endpoint can be used to post the income that was used for the application to SprintHive. 

Endpoint: ```income/v1/incomeVerification/${incomeVerificationId}/confirmation```  
Method: POST  
Body:
```json
{
  "nettIncome": "20000"
}
```

The confirmation is returned in the get income verification endpoint. ```income/v1/incomeVerification/${incomeVerificationId}```

```json
{
  "status": "COMPLETE",
  "incomeConfirmation": {
        "nettIncome": 4300.00
    }
}
```