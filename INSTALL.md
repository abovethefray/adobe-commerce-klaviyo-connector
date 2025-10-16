# Installation and Setup

This guide explains how to install, configure, and deploy this Klaviyo App Builder project using our repository structure.

## 1) Prerequisites
- Node.js 22.x (nvm recommended)
- Adobe I/O CLI: `npm i -g @adobe/aio-cli`
- API Mesh plugin: `aio plugins:install @adobe/aio-cli-plugin-api-mesh`
- Access to Adobe Developer Console (Org → Project → Workspace)
- Adobe Commerce instance (SaaS or PaaS) with API access
- Klaviyo Private API Key and API Revision

## 2) Clone and bootstrap
```bash
git clone <repo-url>
cd klaviyo-app-builder
cp env.dist .env
```

## Setting up aio CLI and API mesh plugin
```bash
npm install -g @adobe/aio-cli 
aio plugins:install @adobe/aio-cli-plugin-api-mesh
```

## Klaviyo account preparation

Follow Klaviyo’s article on [how to create or clone a private API key](https://help.klaviyo.com/hc/en-us/articles/7423954176283) or [find API keys](https://help.klaviyo.com/hc/en-us/articles/115005062267#h_01HRFPP8R1AEVQ744SE33FQTEC) for users with existing keys


Fill `.env` with values (see README for key descriptions). Never commit `.env`.

### Required environment variables (from `.env`)
- Commerce (both flavors)
  - `COMMERCE_BASE_URL` and `COMMERCE_GRAPHQL_ENDPOINT`
    - SaaS: base URL usually includes tenantId and should NOT include `/rest`; GraphQL endpoint is the SaaS GraphQL gateway
    - PaaS: base URL should end with `/rest/V1`; GraphQL endpoint is your instance `/graphql`
- SaaS (Adobe Commerce as a Cloud Service) – IMS OAuth (required for SaaS)
  - `OAUTH_CLIENT_ID`
  - `OAUTH_CLIENT_SECRET`
  - `OAUTH_TECHNICAL_ACCOUNT_ID`
  - `OAUTH_TECHNICAL_ACCOUNT_EMAIL`
  - `OAUTH_ORG_ID`, `OAUTH_SCOPES`
  - Optional: `OAUTH_BASE_URL`, `OAUTH_HOST`
- PaaS/On‑Prem (Commerce Cloud/On‑Premise) – Integration OAuth1 (required for PaaS)
  - `COMMERCE_CONSUMER_KEY`
  - `COMMERCE_CONSUMER_SECRET`
  - `COMMERCE_ACCESS_TOKEN`
  - `COMMERCE_ACCESS_TOKEN_SECRET`
- Klaviyo
  - `KLAVIYO_API_URL` (default `https://a.klaviyo.com/api`)
  - `KLAVIYO_API_REVISION` (e.g., `2025-07-15`)
  - `KLAVIYO_API_KEY` (private API key)
- Product URLs
  - `ACCS_FE_URL` (storefront base URL used to build product links)

#### Flavor-specific notes
- SaaS
  - Prefer IMS OAuth (Server-to-Server) credentials from Adobe Developer Console
  - Ensure `COMMERCE_BASE_URL` matches SaaS requirements (no `/rest` suffix)
- PaaS
  - Use Integration credentials from Commerce Admin → System → Integrations
  - Ensure `COMMERCE_BASE_URL` ends with `/rest/V1` for REST operations

## 3) Configure Adobe Developer Console
1. Go to https://developer.adobe.com/console
2. Create Project → App Builder template
3. Select a Workspace (Dev/Staging/Prod)
4. Add services:
   - I/O Events
   - I/O Management API
   - Adobe I/O Events for Adobe Commerce
   - Adobe Commerce as a Cloud Service
   - API Mesh for Adobe Developer App Builder (add localhost:5000 if needed)
5. Download workspace configuration JSON and save to:
   - `scripts/onboarding/config/workspace.json`

## 4) Link local app to Console project
```bash
aio login
aio console org select
aio console project select
aio console workspace select
aio app use
# choose option 'm' (merge) when prompted
```

## 5) Install dependencies
```bash
npm install
# if peer dependency conflicts occur, you may use: npm install --force
```

## 6) Configure API Mesh (optional but recommended)
- Edit `adobe-api/mesh/mesh.json` (e.g., set storefront origin in `meshConfig.responseConfig`).
- First deploy: `aio api-mesh:create adobe-api/mesh/mesh.json`
- Update: `aio api-mesh:update adobe-api/mesh/mesh.json`
- Status: `aio api-mesh:status`

## 7) Deploy runtime actions
```bash
aio app deploy
```
Verify deployments in Adobe Developer Console → Workspace → Runtime.

## 8) Onboarding (providers and registrations)
```bash
npm run onboard
```
Confirm providers/registrations in Adobe Developer Console.

## 9) Subscribe Commerce instance to events
```bash
npm run commerce-event-subscribe
```
This configures Commerce-side event subscriptions using `.env` and `scripts/onboarding/config/*`.

## 10) UI access (backend UI)
The UI lives in `src/commerce-backend-ui-1/web-src` and is deployed with the actions.
- Dev mode: `aio app dev`
- Deployed: available via the workspace Runtime web URL

## 11) Product catalog sync specifics
- Page/Batch Size controls both fetch size and Klaviyo batch size (max 100; 5MB payload cap).
- Set `ACCS_FE_URL` in `.env` to build product URLs for Klaviyo.

## 12) Storefront Integration Components

To complete the Klaviyo integration for your Adobe Commerce storefront, install the following storefront components. Each repository includes a detailed README with installation and configuration instructions:

- **[Klaviyo Tracking Block for Adobe Commerce Storefront](https://github.com/abovethefray/klaviyo-tracking-block-for-adobe-commerce-storefront)**  
  Enables Klaviyo analytics and tracking on your Adobe Commerce storefront, capturing customer behavior and events.

- **[Klaviyo Checkout Consent Drop-in for Adobe Commerce App](https://github.com/abovethefray/klaviyo-adobe-commerce-app-checkout-consent-dropin)**  
  Provides a checkout consent drop-in component for collecting customer consent to receive marketing communications via Klaviyo.

- **[Klaviyo Signup Block for Adobe Commerce Storefront](https://github.com/abovethefray/klaviyo-signup-block-for-adobe-commerce-storefront)**  
  Adds newsletter signup blocks powered by Klaviyo to your Adobe Commerce storefront, allowing customers to subscribe to marketing lists.

## 13) Troubleshooting
- Timeouts: reduce Page/Batch Size (≤100) and ensure payload < 5MB
- IMS/auth: re-run `aio login` and re-select org/project/workspace
- Commerce API: verify integration credentials and scopes
- API Mesh: validate `mesh.json` and run `aio api-mesh:status`

---
See `README.md` for architecture diagrams, environment variables, and feature overview.
