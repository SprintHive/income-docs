# Setting "Min Income" when Creating an Income Verification Request

When creating an income verification, "minIncome" can be inputted in the inv config. This "minIncome" is the minimum
required income that could be detected with high confidence. This only applies to the primary nett income amount.
Other income does not affect this. 

### Detected nett income is below min income
If the detected income is lower than the min income, the result will be low income confidence (0%). If the detected income is 
equal or higher than the min income, then confidence will be calculated as usual per other methods. 

If "minIncome" is null this behavior will not come into play, because there is no "minIncome" to compare to.

### Pass higher than min income
This comes into play if "minIncome" has been passed in when the income verification was created, and
"passHigherThanMinIncome" is true from tenant config or in the create request config. Then, if the primary detected
nett income is greater than or equal to this minimum required income, the result will be high income confidence (100%).

### Min income and "pass higher than DECLARED income"
When the tenant config "passHigherThanDeclaredIncome" is true (or this is overridden to true in the config when 
creating an income verification), and the detected income is higher than the declared income, the result will be high 
income confidence (100%).

If "minIncome" is inputted when creating the income verification, and the detected nett income is both below the min 
income and above the declared income, then the min income will take precedence and the result will be low income 
confidence (0%). 

## Example body

Endpoint: ```income/v2/incomeVerification```  
Method: POST  
Body:
```json
{
  "config": {
    "minTransactionDaysRequired": 15,
    "minPayslipsDaysRequired": 1,
    "timeOutMinutes": 1440,
    "bankStatementAndPayslipRequired": false,
    "incomeDetectorStrategy": "average",
    "minIncome": "10000",
    "passHigherThanMinIncome": true,
    "passHigherThanDeclaredIncome": true,
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
    "title": "Mr",
    "firstName": "Thomas",
    "surname": "Bayes",
    "idNumber": "1234567890",
    "bankAccountNumber": "1581586785"
  }
}
```
