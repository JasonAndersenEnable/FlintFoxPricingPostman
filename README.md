# RMx Pricing Engine API — Postman Collection

A Postman collection for the Flintfox RMx Pricing Engine API v1.

> **Work in progress** — environment variables and request bodies use placeholder values. Tests are being added incrementally and are not yet present on all requests.

---

## Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Authentication](#authentication)
- [Collection variables](#collection-variables)
- [Collection structure](#collection-structure)
- [Enum reference](#enum-reference)
- [Test scripts](#test-scripts)
- [Newman (CLI)](#newman-cli)
- [Troubleshooting](#troubleshooting)

---

## Overview

This collection covers the RMx Pricing Engine API hosted at:

```
https://ffa-demo.flintfox.com:8005
```

The API is organised into the following functional areas:

| Folder | Purpose |
|---|---|
| `Accrual` | Initialise, step, complete and cancel accrual runs |
| `Cache` | Inspect and serialise the pricing cache |
| `Custom` | Retrieve a custom price for a single line |
| `Diagnose` | Application info, diagnostic settings, and serialised file downloads |
| `Message` | Push data messages to the pricing engine |
| `Notification` | Push change notifications to invalidate cached entities |
| `Price` | Core pricing — single line and batch requests |
| `Setting` | Configure price request serialisation settings |

---

## Prerequisites

- [Postman](https://www.postman.com/downloads/) desktop app or web (v10+)
- Azure AD app registration credentials (client ID, client secret, tenant ID) with access to the Pricing Engine API
- A valid `company_code`, `currency_code`, and `run_type` for your environment
- Newman (optional, for CLI runs): `npm install -g newman`

---

## Setup

1. **Import the collection** into Postman:
   - Open Postman → *Import* → select `RMx_Pricing_Engine_API_postman_collection.json`
2. **Set collection variables** (see [Collection variables](#collection-variables) below):
   - Open the collection → *Variables* tab
   - Fill in values for your target environment
3. Authentication is handled automatically by a pre-request script — no manual token setup required.

> Never commit collection variable values containing real credentials. Keep sensitive values in a local environment file excluded by `.gitignore`.

---

## Authentication

The collection uses **Azure AD OAuth 2.0 client credentials** flow. A pre-request script on the collection root handles token acquisition and refresh automatically.

Before each request, the script:
1. Checks whether a valid `access_token` is already stored and not expired
2. If missing or expired, posts to the Microsoft token endpoint using `client_id`, `client_secret`, and `tenant_id`
3. Stores the returned token and its expiry in collection variables

The token is then passed as `X-API-KEY` on every request via the collection-level auth configuration.

**You only need to set:**

| Variable | Description |
|---|---|
| `tenant_id` | Azure AD tenant ID |
| `client_id` | Azure AD app registration client ID |
| `client_secret` | Azure AD app registration client secret |

The `access_token` and `token_expiry` variables are managed automatically — do not set them manually.

---

## Collection variables

All variables are defined on the collection and can be set under the *Variables* tab. Values marked as sensitive should not be committed.

| Variable | Description | Example | Sensitive |
|---|---|---|---|
| `base_url` | API base URL | `https://ffa-demo.flintfox.com:8005` | No |
| `tenant_id` | Azure AD tenant ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` | Yes |
| `client_id` | Azure AD client ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` | Yes |
| `client_secret` | Azure AD client secret | `your-secret` | Yes |
| `access_token` | Managed automatically by pre-request script | — | Yes |
| `company_code` | Company code for pricing requests | `ACME` | No |
| `currency_code` | Currency code | `USD` | No |
| `run_type` | Pricing run type | `Sales` | No |
| `client_data_source` | Client data source identifier (query param used across most endpoints) | `AX` | No |

---

## Collection structure

```
RMx Pricing Engine API
├── Accrual
│   ├── POST  InitializeAccrualRun
│   ├── POST  InitializeSteppedBreaks
│   ├── POST  CompleteAccrualRun
│   ├── POST  CancelAccrualRun
│   └── POST  GetPriceForeignKeys
├── Cache
│   ├── GET   GetCacheInfo             ← good first connectivity check
│   ├── POST  QueryCache
│   └── GET   SerializeCache
├── Custom
│   └── POST  GetCustomPrice
├── Diagnose
│   ├── GET   GetApplicationInfo       ← best endpoint to verify auth
│   ├── POST  SetDiagnoseSetting
│   ├── GET   DownloadSerializedCache
│   ├── GET   DownloadSerializedHttpRequests
│   └── GET   DownloadSerializedPricingRequests
├── Message
│   ├── POST  AddDataMessage
│   └── POST  AddDataMessages
├── Notification
│   ├── POST  AddChangeNotification
│   ├── POST  AddChangeNotificationSingle
│   ├── POST  AddChangeNotifications
│   └── POST  AddChangeNotificationCollection
├── Price
│   ├── POST  GetPrice
│   ├── POST  PriceSingle
│   ├── POST  PriceRequest             ← always returns 200; errors in body
│   └── POST  PriceCollection
└── Setting
    └── GET   SetPriceRequestsSerializationSettings
```

### Recommended starting points

**Verify connectivity and auth** — run `Diagnose / GetApplicationInfo` first. It requires no body and returns application state. A `200` confirms the token flow is working.

**Check cache health** — run `Cache / GetCacheInfo`. Returns version, sync time, and any errors. Useful before submitting pricing requests to confirm the cache is populated.

**Single line price** — `Price / PriceSingle` or `Price / GetPrice` for a single document line. Populate `company_code`, `run_type`, a product code, and a customer key in the request body.

**Batch pricing** — `Price / PriceRequest` or `Price / PriceCollection` accept an array of requests. Note that these endpoints always return HTTP `200` — errors are embedded in the response body per document, not as HTTP error codes.

---

## Enum reference

Several fields across the collection accept numeric enum values. Reference below:

| Field | Value | Meaning |
|---|---|---|
| `runLevel` | `0` | Incremental |
| `runLevel` | `1` | Full |
| `changeType` | `0` | Insert |
| `changeType` | `1` | Update |
| `changeType` | `2` | Delete |
| `entityType` | `1` | Customer |
| `entityType` | `2` | Product |
| `entityType` | `4` | Vendor |
| `documentType` | `2` | Sales order |
| `requestType` | `0` | Price only |
| `priceRequestSource` | `4` | Price inquiry |
| `cacheObjectType` | `0` | All |
| `cacheObjectType` | `1` | Company |
| `cacheObjectType` | `2` | Component |
| `cacheObjectType` | `9` | Trade agreement version |
| `cacheObjectType` | `14` | Trade agreement price |
| `cacheObjectType` | `19` | Price model |
| `messageStatus` | `0` | Created |

---

## Test scripts

> Tests are being added incrementally. Not all requests have test coverage yet.

Requests with test scripts currently validate:

| Check | Detail |
|---|---|
| Status code | `200` expected |
| Response time | Under 2000ms |
| Schema | Required fields present and correctly typed |
| Pricing integrity | `netPriceExtended` = `netPrice × quantity` |
| Error consistency | `errorMessage` and `errorLabelFormatters` agree |
| Price levels | Successful responses contain at least one list and one net level |
| Sync status | `isLatest: true` paired with empty `error` field |

Test results are visible in the **Test Results** tab after sending a request, or in the Collection Runner summary.

---

## Newman (CLI)

Run the collection from the command line using Newman:

```bash
# Install Newman
npm install -g newman

# Run the collection
newman run RMx_Pricing_Engine_API_postman_collection.json \
  --reporters cli,json \
  --reporter-json-export results.json
```

> Because sensitive values are stored as collection variables (not in a separate environment file), ensure you export the collection with values populated only in a secure, non-committed copy for CI use.

### CI/CD (GitHub Actions example)

```yaml
- name: Run Pricing Engine tests
  run: |
    newman run RMx_Pricing_Engine_API_postman_collection.json \
      --reporters cli,junit \
      --reporter-junit-export newman-results.xml

- name: Upload test results
  uses: actions/upload-artifact@v4
  with:
    name: newman-results
    path: newman-results.xml
```

Store `tenant_id`, `client_id`, and `client_secret` as GitHub Actions secrets and inject them via a pre-run script or environment substitution rather than committing values.

---

## Troubleshooting

**401 Unauthorized**
The token acquisition failed. Check that `tenant_id`, `client_id`, and `client_secret` are set correctly in the collection variables. Open the Postman Console (`View → Show Postman Console`) to inspect the pre-request script output.

**All prices returning `0` with no HTTP error**
The `PriceRequest` and `PriceCollection` endpoints always return `200`. Check the `errorMessage` field inside each document in the response array — an unresolvable `custVendKey` or `companyKey` of `0` will silently return zero prices.

**`Invalid customer` in errorMessage**
The customer key in the request body is not resolving. Verify the `custVendKey` and `companyKey` values match a known customer in the target environment.

**Cache appears stale or empty**
Run `Cache / GetCacheInfo` to check `isLatest` and `syncDateTime`. If the cache is out of date, the sync job may still be running or may have failed. Use `Notification / AddChangeNotification` to trigger a cache invalidation for a specific entity if needed.

**`clientDataSource` — what value should this be?**
Many endpoints accept a `clientDataSource` query parameter identifying the source ERP system. Set the `client_data_source` collection variable to the value provided for your environment (e.g. `AX` for Dynamics AX / 365 Finance).

---

## Contributing

1. Branch from `main`
2. Make your changes to the collection in Postman
3. Export the updated collection: *File → Export → Collection v2.1*
4. Commit the exported JSON — do not include populated variable values
5. Open a pull request with a description of what changed and why

---

## License

Copyright © [Year] [Your Company Name]. All rights reserved.
Internal use only. See `LICENSE` for details.
