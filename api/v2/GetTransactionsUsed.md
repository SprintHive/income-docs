# Retrieving enriched transactions

There are two options described below for retrieving transactions.

## Transactions
This endpoint will return the consolidated list of transactions that were used to determine income. All transaction 
enrichments that are enabled in your environment will also be returned in this response.

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
      "transactionValue": -7.90,
      "inPrimaryIncomeStream": false,
      "inOtherIncomeStream": false
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

### Detected income
Transactions that were selected as the primary income will be identified with `"inPrimaryIncomeStream": true`. 
If "otherIncome" was declared when creating the income verification, transactions that were selected as that other 
income will be identified with `"inOtherIncomeStream": true`. All other transactions that were not identified as income 
will have these fields be false, as in the example above.

## Enriched Transactions
This endpoint will return the consolidated list of transactions that were used to determine income. Additional 
transaction enrichments that are not enabled in general in your environment, can also be requested in the request body.

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
      "inPrimaryIncomeStream": false,
      "inOtherIncomeStream": false,
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

## Download Transactions as a csv file
This endpoint will download the consolidated list of transactions that were used to determine income as a csv file (opens in Excel). 
All transaction enrichments that are enabled in your environment will also be returned in this file.
The file will have the name "transactions-{incomeVerificationId}.csv"

Endpoint: ```/v2/incomeVerification/{incomeVerificationId}/transactions/download/csv```  
Method: GET

Example response:
file name: transactions-8e55aa90-8afe-444c-9c6a-686f36c70e3f.csv
The table below represents the contents of the csv file:

| Date       | Description                                                       | Amount   | Balance  | Category          | Transaction Type | Third Party | Third Party Sub Division      | Tags              | Failure Indicator | In Primary Income Stream | In Other Income Stream |
|------------|-------------------------------------------------------------------|----------|----------|-------------------|------------------|-------------|-------------------------------|-------------------|-------------------|--------------------------|------------------------|
| 2026-05-20 | Live Better Round-up Transfer                                     | -2.85    | 1614.98  | Automatic Savings | Mobile Transfer  | Capitec     | Live Better Round-up Transfer | Automatic Savings |                   | FALSE                    | FALSE                  |
| 2026-05-22 | Arthur Ford Goldenwalk Germiston (Card 1234)                      | -100.00  | 1514.98  | Car Repayment     | Card             | Ford        | Ford                          | Car Repayment     |                   | FALSE                    | FALSE                  |
| 2026-05-22 | Payment Received: Mamathabi Payment 2678380750                    | 420.00   | 1934.98  |                   | Direct Payment   |             |                               | Direct Payment    |                   | FALSE                    | FALSE                  |
| 2026-05-22 | Banking App External Payment: Sars                                | -102.00  | 1832.98  | Tax               | Mobile Payment   | SARS        | SARS                          | Tax               |                   | FALSE                    | FALSE                  |
| 2026-05-25 | Payment Received: Nhighere63 63 Pay3161821900470salary 2728392253 | 17011.26 | 18844.24 | Salary/Wages      | Direct Payment   |             |                               | Salary/Wages      |                   | TRUE                     | FALSE                  |
