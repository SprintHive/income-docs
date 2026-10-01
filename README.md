# income-docs
Documentation for the Income Verification service

## General

- [Security](guides/security/CreatingJsonWebToken.md) — Creating a JSON Web Token and using your API key
- [Quick Start](guides/quick-start-guide/README.md) — The simplest end-to-end integration, with a Postman collection
- [OpenAPI spec v2](openApi/openapi-public-v2.yaml) — Full v2 API contract, including error codes ([v1](openApi/openapi-public-v1.yaml))
- [Notifications](guides/notifications/Notifications.md) — Document processed and status changed notifications over webhook or SQS 
- [FAQ](FAQ.md) — Frequently asked questions

## Case Lifecycle

An income case is asynchronous and depends on the documents it receives. It stays open
while documents arrive and are processed, until the document rules are met or the case times out.

1. **[Create](#case-creation)** a case. The `config` sets the document requirements, timeout (`timeOutMinutes`), personal details of the applicant and other optional configs.
2. **[Submit documents](#document-submission)** by API upload or as extracted data, or have the applicant send them by email or through the upload site.
3. **Processing.** Each document is classified, has its data extracted, receives tampering checks, and validation checks against the document requirements set on creation. Once all checks pass then income detection runs.
4. **[Track progress](#tracking-progress-and-results)** with the case state, document chase and notifications. The case stays `IN_PROGRESS` until it has enough valid documents or it times out.
5. **[Outcome](#tracking-progress-and-results).** The case ends as `SUCCESS` (if document validation passed and income was detected), `FAILED` or `CONFIRMED_FRAUD`. An agent or third party system can also [capture income manually](#manual-review).
6. **[Confirm](#post-case-completion-feedback)** the income that was used for the application.
7. **[Retention](#data-retention).** Once the retention period has passed the case is deleted, and later calls for it return `410 Gone`.

See [what the statuses mean](api/v2/GetIncomeVerificationState.md#what-do-the-different-statuses-mean) for the full status and sub-status table.

## API Reference

### Case Creation

| Endpoint | Description |
|----------|-------------|
| [Create Income Verification Request](api/v2/CreateIncomeVerificationRequest.md) | Create a new income verification case |
| [Disable Fraud Checks](api/v2/DisableFraudForTesting.md) | Config option: disable fraud checks in testing environments |
| [Min Income](api/v2/MinIncome.md) | Config option: minimum primary nett income required for high confidence |
| [Bank Statement and Payslip Required](api/v2/IncomeFromBankStatementAndPayslip.md) | Config option: require both a bank statement and a payslip income result |

### Document Submission

| Endpoint | Description |
|----------|-------------|
| [Upload Document](api/v2/UploadDocument.md) | Upload a bank statement or payslip file to a case |
| [Upload Extracted Bank Statement Data](api/v2/UploadExtractedBankStatementData.md) | Submit bank statement transactions that have already been extracted |
| [Upload Document by Correlation Id](api/v2/UploadDocumentByCorrelationId.md) | Upload a file to the case(s) matching your correlationId |

### Tracking Progress and Results

| Endpoint | Description |
|----------|-------------|
| [Get Income Verification](api/v2/GetIncomeVerification.md) | Retrieve the case as it was created: config, declared income and applicant details |
| [Get Income Verification State](api/v2/GetIncomeVerificationState.md) | Retrieve the case status, linked documents and income result |
| [Document Chase](api/v2/DocumentChase.md) | List the income documents received and those still outstanding |
| [Notifications](guides/notifications/Notifications.md) (guide) | Document processed and status changed notifications over webhook or SQS |

### Document Data Retrieval

| Endpoint | Description |
|----------|-------------|
| [Get Extracted Document Data](api/v2/GetExtractedDocumentData.md) | Retrieve the data extracted from a document |
| [Download Document](api/v2/GetDocumentContent.md) | Download the original document file |
| [Get Transactions Used](api/v2/GetTransactionsUsed.md) | Retrieve the enriched transactions used to determine income |
| [Earning and Deduction Types](api/v2/EarningAndDeductionTypes.md) | Reference: payslip earning and deduction types |

### Manual Review

| Endpoint | Description |
|----------|-------------|
| [Manual Capture Income](api/v2/ManualCaptureIncome.md) | Capture income after manually reviewing the documents |

### Post Case Completion Feedback

| Endpoint | Description |
|----------|-------------|
| [Income Confirmation](api/v2/IncomeConfirmation.md) | Send the confirmed income and application result back to SprintHive |

### Case Management

| Endpoint | Description |
|----------|-------------|
| [Filter Income Verifications](api/v2/IncomeFiltering.md) | Filter cases by field values |

### Data Retention

| Endpoint | Description |
|----------|-------------|
| [Data Retention and Deletion](guides/data-retention/DataRetention.md) (guide) | When cases are deleted and how the `410` entity-gone response looks |

## Legacy (v1)

| Endpoint | Description |
|----------|-------------|
| [Migrating from v1 to v2](guides/migrate-v1-to-v2/README.md) (guide) | Changes needed to move from v1 to v2 |
| [Income Confirmation (v1)](api/v1/IncomeConfirmation.md) | Send the confirmed income (v1) |
| [Income Filtering (v1)](api/v1/IncomeFiltering.md) | Filter cases by field values (v1) |

## Release Notes

| Release | Date | Summary |
|---------|------|---------|
| [v2.5.3](releaseNotes/release-v2.5.3.md) | 6 Feb 2025 | Additional document classification types |
| [v2.3.13](releaseNotes/release-v2.3.13.md) | 1 May 2024 | Income filtering and provisional loan amount |
| [v2.3.9](releaseNotes/release-v2.3.9.md) | 1 Mar 2024 | New `CLASSIFICATION_FAILED` document status |
