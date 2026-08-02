---
name: Upload anonymized revenue data for indicative offers
description: >-
  As a Wayflyer partner, upload anonymized merchant revenue data (batch or
  file) so Wayflyer can compute personalised indicative funding offers that
  surface in the CTA.
api: openapi/wayflyer-embedded-finance-openapi-original.json
operations:
  - wf_embedded_finance_service_apps_web_core_views_partner_token_create_partner_token
  - wf_embedded_finance_service_apps_web_anonymous_data_upload_views_anonymous_upload_initiate_batch_upload
  - wf_embedded_finance_service_apps_web_anonymous_data_upload_views_anonymous_upload_initiate_file_upload
  - wf_embedded_finance_service_apps_web_anonymous_data_upload_views_anonymous_upload_complete_file_upload
  - wf_embedded_finance_service_apps_web_anonymous_data_upload_views_anonymous_upload_get_status
---

# Upload anonymized revenue data for indicative offers

Partner-scope flow — authenticate with the **partner token** (not a company
token). Base URL: `https://api.wayflyer.com/financing/`.

## Steps

1. **Get a partner token.** `POST /partner-token/`
   (`..._partner_token_create_partner_token`); cache ~24h.
2. **Pick the upload mode.**
   - Small/medium datasets (recommended): `POST /partner/anon-data-upload/batch/`
     (`..._anonymous_upload_initiate_batch_upload`) with a JSON array of
     company records.
   - Large datasets: `POST /partner/anon-data-upload/file/`
     (`..._anonymous_upload_initiate_file_upload`), upload the parts, then
     finish with `PATCH /partner/anon-data-upload/upload/{upload_id}/`
     (`..._anonymous_upload_complete_file_upload`).
3. **Shape each record.** Include at least one revenue metric, in priority
   order: `annual_revenue` (preferred), or `revenue_in_period` +
   `num_period_months`, or `shipping_in_period` + `num_period_months`.
   Add `company_id` (same anonymous id used for company tokens), `currency`
   (ISO 4217), `country` (ISO 3166-1 alpha-2), optional `state` (ISO 3166-2),
   `incorporation_date`, `client_onboarding_date`.
4. **Track processing.** `GET /partner/anon-data-upload/upload/{upload_id}/`
   (`..._anonymous_upload_get_status`).
5. **Repeat weekly.** Weekly cadence is recommended; fresher data improves
   offer accuracy for fast-growing merchants.

## Rules

- Data must be anonymized — `company_id` must not reveal merchant identity
  without consent.
- 400 on `PATCH .../upload/{upload_id}/` means an incomplete/invalid file
  upload — check the `{detail}` message.
- Respect `Retry-After` on 429 (sliding-window limits vary per endpoint).
