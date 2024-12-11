# Retrieving enriched transactions

There are two options described below for retrieving transactions.

## Transactions
This endpoint will return the consolidated list of transactions that were used to determine income. Any transaction 
enrichments that were enabled when the income verification case was created, will also be returned in this response.

Endpoint: ```/v2/incomeVerification/{incomeVerificationId}/transactions```  
Method: GET
Example response:

```json
{
  "transactions": [
    {
      "id": "62042230-f2f5-400f-8589-480b7d315662",
      "date": "2022-03-20",
      "description": "FEE-POS DECLINED INSUFF FUNDS Checkers Long beach 5709",
      "balance": 12.45,
      "transactionValue": -7.90
    },
    ...
  ],
  "creationDate": "2024-01-12T06:51:33.348899",
  "name": "Jane Soap",
  "address": "ACME Pty (Ltd)\n23 Brick Lane\nACME Town - 9999",
  "bankAccount": {
    "accountNumber": "9999999999",
    "bank": "Absa Bank"
  }
}
```

## Enriched Transactions
This endpoint will return the consolidated list of transactions that were used to determine income. Additional 
transaction enrichments to be fetched, that may not have been enabled when the income verification case was created, 
can also be included in the request body.

Use of this endpoint may have billing implications. Contact SprintHive if you would like to enable it. 

Endpoint: `/v2/incomeVerification/{incomeVerificationId}/enrichedTransactions`\
Content Type: `application/json`\
Request Method: `POST`
Request body: The selections of enrichments to return with the transactions 

Example body:

```json
{
  "transactionCategory" : true,
  "transactionThirdParty" : true,
  "transactionType" : true,
  "transactionFailureIndicator" : true
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


Example response:

```json
{
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
      "tags": [
        "Groceries"
      ]
    },
    ...
  ],
  "creationDate": "2024-01-12T06:51:33.348899",
  "name": "Jane Soap",
  "address": "ACME Pty (Ltd)\n23 Brick Lane\nACME Town - 9999",
  "bankAccount": {
    "accountNumber": "9999999999",
    "bank": "Absa Bank"
  }
}
```
