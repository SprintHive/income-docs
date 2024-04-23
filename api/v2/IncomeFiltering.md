## Filtering Income Verification Requests

Endpoint: `/v2/incomeVerification/filter`\
Content Type: `application/json`\
Request Method: `POST`

Example payload:

```json
{
  "filters": [
    {
      "field": "",
      "values": [
        ""
      ],
      "matchPhrase": false
    }
  ],
  "dateField": "",
  "page": 0,
  "pageSize": 0,
  "sortDirection": "",
  "sortFields": [
    ""
  ],
  "fromDate": "2023-01-10T00:00:00.000Z",
  "toDate": "2023-01-10T00:00:00.000Z"
}
```

| **Field**     | **Values**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | **Defaults**                                                                                               |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|
| filters       | A list of filters. A filter contains a field name. Valid filterable fields are `createdDate, statusV2, subStatusV2` Each filter can contain one or many values to filter on.<br>`matchPhrase` - flag to indicate whether to search "filtered value" as individual words, or search for the entire phrase appearing as is. Defaults to `true`<br>Valid `statusV2` fields are `inProgress, referredToFraud, confirmedFraud, failed, success`<br>Valid `subStatusV2` fields are `WaitingForDocuments, ProblemsWithDocuments, DataExtractionFailed, NoIncomeDetected, LowIncomeConfidence, SuspectedDocumentFraud, SuspectedCaseFraud, ConfirmedDocumentFraud, ManualConfirmedFraud, ConfirmedCaseFraud, HighIncomeConfidence, ManuallyCaptured` | Empty filter list, so return all values.                                                                   |
| dateField     | Field to use for date range filter. Used with `fromDate` and `toDate`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Validation error if specified field does not exist or is not a date type field, or`createdDate` if omitted |
| fromDate      | Date range filter start date in a `2023-01-10T00:00:00.000Z` format                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Beginning of time.<br>Validation error if set to after `toDate` or the current date.                       |
| toDate        | Date range filter start date in a `2023-01-10T00:00:00.000Z` format                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Current time.<br>Validation error if set to before `fromDate`.                                             |
| page          | Page number of the result set to return                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `0` if omitted.                                                                                            |
| pageSize      | Page size of the result set to return                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | `10` if omitted.<br>Validation error if set to more than `200` .                                           |
| sortDirection | One of  `asc` or `desc`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `desc`                                                                                                     |
| sortFields    | Sortable fields - any of `incomeVerificationId, createdDate, correlationId, idNumber, emailAddress, surname`.<br>Sort priority is in the order the fields appear in the list.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | `createdDate, incomeVerificationId`                                                                        |

Example response:

```json
{
  "numberOfElements": 2,
  "first": true,
  "number": 0,
  "size": 10,
  "content": [
    {
      "incomeVerificationId": "28042230-f2f5-400f-8589-480b7d315694",
      "correlationId": "28042230-f2f5-400f-8589-480b7d315694",
      "createdDate": "2024-04-05T17:56:17.048",
      "status": "SUCCESS",
      "subStatus": "HIGH_INCOME_CONFIDENCE"
    },
    {
      "incomeVerificationId": "28042230-f2f5-400f-8589-480b7d315694",
      "correlationId": "28042230-f2f5-400f-8589-480b7d315694",
      "createdDate": "2024-04-04T11:26:22.115",
      "status": "FAILED",
      "subStatus": "NO_INCOME_DETECTED"
    }
  ],
  "sort": {
    "sorted": true,
    "unsorted": false,
    "empty": false
  },
  "pageable": {
    "offset": 0,
    "pageNumber": 0,
    "sort": {
      "sorted": true,
      "unsorted": false,
      "empty": false
    },
    "paged": true,
    "pageSize": 10,
    "unpaged": false
  },
  "last": true,
  "empty": false
}
```