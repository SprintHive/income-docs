# Create Income Verification Request

This endpoint is used to create an income verification request.  

Endpoint: ```income/v2/incomeVerification```  
Method: POST  
Body:
```json
{
  "config": {
    "minTransactionDaysRequired": 15,
    "minPayslipsDaysRequired": 1,
    "timeOutMinutes": 1440,
    "limitTransactionsByCutOffDate": false,
    "determineIncomeOnTimeOut": false,
    "bankStatementAndPayslipRequired": false,
    "incomeDetectorStrategy": "average",
    "maxAgeDays": {
      "bank-statement": 5000,
      "payslip": 5000
    }
  },
  "correlationId": "test-1",
  "primaryIncome": {
    "nettIncome": "20000",
    "grossIncome": "25000"
  },
  "applicantDetails": {
    "dateOfBirth": "",
    "emailAddress": "",
    "currentEmployerName": "",
    "currentEmployeeNumber": "",
    "title": "Mr",
    "initials": "",
    "firstName": "Thomas",
    "surname": "Bayes",
    "idNumber": "1234567890",
    "passportNumber": "",
    "taxNo": "",
    "bankAccountNumber": "1581586785",
    "bankBranchCode": "",
    "bankName": ""
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

## Configuration Options

The behaviour of the service can be configured by inputs provided when creating the request.
The following section describes each configuration option in more detail.

### timeOutMinutes

How long (in minutes) the service should wait for data to determine income from before giving up and declaring the 
income unverified. This is checked regularly in INV and may result in multiple cases being timed out for a particular check.

### minTransactionDaysRequired

The number of days of bank statement transactions we need to complete this verification 
(e.g. 65 to ensure that 3 months' income events are captured taking occasional variations of pay dates due to 
weekends/holidays into account)

### bankStatementAndPayslipRequired

If the verification requires both a bank statement and a payslip (true) or just one of the two (false) in order to 
completely process a case

### maxAgeDays.bank-statement

Maximum allowed age of a bank statement in days, if older will be ignored

### maxAgeDays.maxAgeDays.payslip

Maximum allowed age of a payslip in days, if older will be ignored

### correlationId

The calling system's unique identifier for this income request. Typically, the customer's application ID. 
This will be used to link back to your system if you use push notification via the web hook

### declaration.grossIncome

The applicant's gross income as per their payslip

### declaration.nettIncome

The applicant's net take home pay as per their payslip or bank statement

### declaration.payCycleInDays

How often the applicant gets paid (e.g. 7 for weekly, 14 for fortnightly, 30 for monthly)

> Defaults to 30

### incomeDetectorStrategy

Different income detector strategies can be used depending on how the applicant is employed.  

> Defaults to average

The following strategies are available by default:

 - min - When using multiple months the system will find the consistent income steam over the selected months and use the smallest amount  
 - average -When using multiple months the system will find the consistent income steam over the selected months and use the average of the amounts found  
 - max - When using multiple months the system will find the consistent income steam over the selected months and use the largest amount  
 - irregular - The system will add up all the income transactions excluding specific income transactions, like loans received, and calculate the average of the monthly totals  