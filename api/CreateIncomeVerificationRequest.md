# Create Income Verification Request

This endpoint is used to create an income verification request.

Endpoint: ```income/v2/incomeVerification```  
Method: POST  
Body:
```json
{
  "correlationId": "test-1",
  "primaryIncome": {
    "nettIncome": "20000",
    "grossIncome": "25000"
  }
}
```

Response:
```json
{
  "incomeVerificationId": "49f9290c-d393-4af4-8072-0400303f64c8",
  "correlationId": "test-1"
}
```
