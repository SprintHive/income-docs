# Release Notes for v2.5.3

Date: 6 Feb 2025

## Introducing additional document classification types
### Already present Document Types
- BANK_STATEMENT
- PAYSLIP
- MULTI_DOC
- UNKNOWN

### Additional Document Types
- PROOF_OF_ACCOUNT
- ID_BOOK
- ID_CARD_FRONT
- ID_CARD_REVERSE
- ID_CARD_FRONT_AND_REVERSE
- DRIVING_LICENCE_FRONT
- DRIVING_LICENCE_REVERSE
- DRIVING_LICENCE_FRONT_AND_REVERSE
- COB

Expect to see these document types within the [get Income Verification State](../api/v2/GetIncomeVerificationState.md) response.

The docs can be found here under `DocumentType`:

[openapi-public-v2.yaml](../openApi/openapi-public-v2.yaml)
