---
name: Render a Wayflyer funding offer and run the embedded application
description: >-
  Authenticate as a partner, mint a company token for a merchant, fetch the
  CTA banner state, and drive the v5 embedded funding application from start
  to submit against the Wayflyer Embedded Finance API.
api: openapi/wayflyer-embedded-finance-openapi-original.json
operations:
  - wf_embedded_finance_service_apps_web_core_views_partner_token_create_partner_token
  - wf_embedded_finance_service_apps_web_core_views_company_token_create_company_token
  - wf_embedded_finance_service_apps_web_cta_views_banner_get_banner
  - wf_embedded_finance_service_apps_web_cta_views_banner_dismiss_banner
  - wf_embedded_finance_service_apps_web_embedded_application_views_embedded_application_start_embedded_application
  - wf_embedded_finance_service_apps_web_embedded_application_views_embedded_application_get_embedded_application
  - wf_embedded_finance_service_apps_web_embedded_application_views_embedded_application_submit_embedded_application
---

# Render a Wayflyer funding offer and run the embedded application

Base URL: `https://api.wayflyer.com/financing/` (sandbox: `https://sandbox-api.wayflyer.com/financing/`).
All requests are JSON over HTTPS with `Authorization: Bearer <token>`.

## Steps

1. **Get a partner token (backend only).** `POST /partner-token/`
   (`..._partner_token_create_partner_token`) with
   `{"partner_id": "<client_id>", "partner_secret": "<client_secret>"}`.
   Cache the JWT for its `expires_in` (86000s ≈ 24h). Never expose the
   client secret or this call to the frontend.
2. **Mint a company token for the merchant.** `POST /partner/company-token/`
   (`..._company_token_create_company_token`) with the partner token and
   `{"company_id": "<anonymous-id>", "user_id": "<user-id>"}`. `company_id`
   must be an anonymous, stable string under 255 chars.
3. **Fetch the CTA.** `GET /company/cta/` (`..._banner_get_banner`) with the
   company token. Handle every `BannerState`: `indicative_offer`,
   `generic_offer`, `continue_application`, `continue_embedded_application`
   — and `null` (ineligible or in a dismissal cooling-off; do not render).
   Use the Wayflyer-managed copy in `data.config` (`text`, `button_label`,
   `bullet_points`).
4. **Respect dismissals.** On user dismissal call `POST /company/cta/dismiss/`
   (`..._banner_dismiss_banner`); repeated dismissals back off exponentially.
5. **Start the application.** `POST /company/embedded-application/`
   (`..._start_embedded_application`). Poll state with
   `GET /company/embedded-application/` (`..._get_embedded_application`).
6. **Submit.** `POST /company/embedded-application/submit/`
   (`..._submit_embedded_application`). Handle the documented
   `SubmitEmbeddedApplicationErrorCode` values: `details_incomplete`,
   `no_open_application`, `open_hosted_application_exists`,
   `duplicate_company`, `ineligible_application`,
   `registration_or_documents_required`.

## Rules

- No idempotency keys exist — rely on the server-side state guards
  (409 Conflict / 423 Locked) and never blind-retry writes
  (see `conventions/wayflyer-conventions.yml`).
- On 429, respect `Retry-After`; watch `RateLimit-Remaining`.
- Errors are `{detail}` or `{error_code, detail}` — catalog in
  `errors/wayflyer-problem-types.yml`.
- Test first in the sandbox with `isSandbox`-mode credentials and the
  simulation endpoints (see `sandbox/wayflyer-sandbox.yml`).
