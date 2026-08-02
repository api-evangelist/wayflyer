---
name: Complete application details and upload supporting documents
description: >-
  Fill in the applicant's user and company details and manage KYC/supporting
  document uploads for a Wayflyer funding application (headless integrations
  that own the application UI).
api: openapi/wayflyer-embedded-finance-openapi-original.json
operations:
  - wf_embedded_finance_service_apps_web_core_views_user_details_get_user_details
  - wf_embedded_finance_service_apps_web_core_views_user_details_update_user_details
  - wf_embedded_finance_service_apps_web_core_views_company_details_get_company_details
  - wf_embedded_finance_service_apps_web_core_views_company_details_update_company_details
  - wf_embedded_finance_service_apps_web_core_views_company_details_get_business_types
  - wf_embedded_finance_service_apps_web_core_views_industry_classification_search_industry_classifications
  - wf_embedded_finance_service_apps_web_core_views_document_upload_create_document_upload_url
  - wf_embedded_finance_service_apps_web_core_views_document_upload_confirm_document_upload
  - wf_embedded_finance_service_apps_web_core_views_document_upload_list_document_uploads
  - wf_embedded_finance_service_apps_web_core_views_document_upload_delete_document_upload
---

# Complete application details and upload supporting documents

Company-scope flow — authenticate every call with the merchant's
**company token**. Base URL: `https://api.wayflyer.com/financing/`.

## Steps

1. **Read current state.** `GET /company/user-details/`
   (`..._get_user_details`) and `GET /company/company-details/`
   (`..._get_company_details`) — both 404 until first written.
2. **Populate reference values.** Fetch valid business types with
   `GET /company/company-details/business-types/` (`..._get_business_types`);
   resolve industry codes with
   `GET /company/industry-classification/search/`
   (`..._search_industry_classifications`, params `query` + `max_results`
   1-100) — an `invalid_industry_code` error on save means the code did not
   come from this search.
3. **Write details.** `PUT /company/user-details/` (`..._update_user_details`)
   and `PUT /company/company-details/` (`..._update_company_details`).
   Handle 422 (validation), 409 `duplicate`, and the shared codes
   `too_many_revisions` and `application_already_submitted` (423 Locked —
   stop editing after submission).
4. **Upload documents.** `POST /company/documents/`
   (`..._create_document_upload_url`) to get an upload URL, PUT the file
   bytes to it, then `POST /company/documents/{document_id}/confirm/`
   (`..._confirm_document_upload`). List with `GET /company/documents/`
   (`..._list_document_uploads`, `include_unconfirmed=true` to see pending);
   remove mistakes with `DELETE /company/documents/{document_id}/`
   (`..._delete_document_upload` — 423 when the document is locked).

## Rules

- Documents follow a create-URL -> upload -> confirm pattern; an unconfirmed
  upload is invisible to the application until confirmed.
- No idempotency keys — treat 409/423 as authoritative state, not transient
  errors (see `conventions/wayflyer-conventions.yml`).
- Error envelope and code enums: `errors/wayflyer-problem-types.yml`.
