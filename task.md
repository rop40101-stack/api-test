# API FORGE — Full Project Task Specification

> **Purpose:** Build a real, secure, mobile-friendly AI API testing website. This file is the source of truth for implementation. Follow it end-to-end; do not stop after making a design mockup or writing an architecture proposal.

## 1. Product goal

Create **API FORGE — AI API Testing & Diagnostics**, a developer tool where users can configure AI API providers, send real requests, discover models, inspect responses, measure latency, and diagnose errors.

The application must run locally with ordinary Node.js/npm commands and be deployable to a standard Node.js host. **Do not use Docker.** It must work well on Android-sized screens and low-powered devices.

## 2. Required technology

- Frontend: React, Vite, TypeScript, React Router.
- Styling: Tailwind CSS plus small custom CSS as needed.
- Backend: Node.js, Express, TypeScript.
- Validation: Zod (or equivalent runtime schema validation).
- HTTP: Fetch or provider-appropriate official SDKs where useful.
- Tests: Vitest and suitable route/provider unit tests.
- Use maintained, compatible package versions. Avoid unnecessary dependencies.
- Include root-level npm scripts for development, build, start, and tests.

Use separate client and server modules. Add a root README, `.env.example`, and `.gitignore`.

## 3. Visual design and accessibility

Use a polished, minimalist futuristic developer-tool design:

- Black, white, charcoal, and neutral borders; limited accent colors.
- Orbitron for selected technical headings and Space Grotesk for normal UI text, with system-font fallbacks.
- Compact cards and forms, crisp separators, subtle animation, readable code/JSON panels.
- Desktop sidebar; mobile navigation drawer or compact bottom navigation.
- Do not use huge empty panels, excessive gradients, constant animations, or decorative elements that reduce space for the actual work.
- Empty result panels should stay compact until a test is run.
- Include clear focus indicators, accessible labels, keyboard operation, responsive layouts, and adequate contrast.
- Use real status colors only for real outcomes. Do not show fabricated metrics or successes.

Brand name: **API FORGE**. Subtitle: **AI API Testing & Diagnostics**. A simple text-based logo is sufficient.

## 4. Main navigation/pages

Implement these working pages:

1. Dashboard
2. API Playground
3. Providers & Connections
4. Model Explorer
5. General API Health Check
6. **Google AI Studio Health Check — a separate, dedicated page**
7. Request Builder
8. Response Inspector
9. Request History
10. Settings
11. Documentation & Troubleshooting

The Google AI Studio page must have its own route, controls, check workflow, loading/error states, and report view. It must not be just a button or subsection inside the generic health checker.

## 5. Dashboard

Show useful, real application/session metrics: requests run, successful and failed requests, average latency, configured provider count, recently tested models, and recent sanitized errors. Metrics must be calculated from actual recorded events; show an honest empty state before tests have run. Provide quick links to the Playground and Google AI Studio Health Check.

## 6. Provider profiles and configuration

Support create, edit, duplicate, delete, test, save, import, and export provider profiles. A non-secret profile may store a name, provider format, base URL, path, model ID, timeout, and safe defaults.

### Required fields

- Provider display name
- Provider format/adapter
- API base URL and endpoint/path
- API key or credential for the current request
- Model ID
- HTTP method where relevant
- Authentication mode
- Request format
- Timeout
- Maximum output tokens where the provider supports it

### Advanced optional fields

- Custom headers (validated and secret-redacted in UI/logs)
- Custom JSON body
- System instruction
- Temperature and top-p where supported
- Streaming toggle where supported
- Query parameters where appropriate (never place credentials in query strings)
- Retry preferences
- Provider-specific generation options

Only display compatible parameters for each adapter. Do not imply that unsupported settings work. Preserve user edits when changing a provider; if a change would reset fields, warn or ask first.

### Provider formats to implement

Implement separate adapters and normalize their responses for the common UI:

1. OpenAI-compatible Chat Completions API
2. OpenAI Responses API
3. Anthropic Messages API
4. Google Gemini native REST API
5. Custom REST/JSON provider with user-specified method, endpoint, headers, and request body

Every adapter must handle its own authentication, request schema, response extraction, errors, and (where supported) streaming. Similar-looking APIs are not guaranteed to be interchangeable. Document limitations instead of claiming universal compatibility.

### Provider actions

All visible controls must work: Create, Edit, Duplicate, Delete, Test Connection, Test Model, Save Profile, Reset, Import, Export. Confirm destructive deletion. Export/import must never include secrets.

## 7. API Playground and Request Builder

Provide controls for provider profile, API format, URL/path, model ID, API key, system instruction, user prompt, supported generation settings, timeout, and streaming if implemented. Include an editable JSON request body with validation and formatting.

Working actions: **Send Request**, Stop/Cancel when supported, Clear Input, Format JSON, Copy Request, Copy Response, Export redacted request diagnostics, Reset Parameters.

The frontend must call the app backend. The backend must make the actual upstream request. Do not simulate successful API responses or embed mock responses in the production flow.

Handle loading, cancellation, timeout, invalid JSON, unavailable models, unsupported parameters, network failures, malformed response bodies, and upstream errors without crashing the UI.

## 8. Dedicated Google AI Studio Health Check

Create a standalone page named **Google AI Studio Health Check**.

### 8.1 Connection controls

- Google Gemini API key input, masked by default, with show/hide control
- Model selector populated dynamically from the API when possible
- Manual model ID entry
- API version/base URL display (default Gemini REST base: `https://generativelanguage.googleapis.com/v1beta`)
- Test API Key / credential access
- Fetch Available Models / refresh
- Test Text Generation
- Run Full Health Check
- Clear Credentials

Do not hardcode a model ID as the only choice. If a default suggestion is displayed, label it as a suggestion and verify it by live discovery or a real request. A model name appearing in documentation is not proof that it is accessible to this user's key.

### 8.2 Gemini model discovery

Use the official ListModels REST operation:

`GET https://generativelanguage.googleapis.com/v1beta/models`

Send the credential using the `x-goog-api-key` header. Parse the real response and show available metadata such as model ID/name, supported generation methods, and token limits if those fields are present. Offer search, refresh, and manual entry. Filter to models that support `generateContent` where capability metadata is available.

### 8.3 Gemini text generation

Use the official `generateContent` endpoint:

`POST https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent`

Replace `{model}` with a validated model name from discovery or user-entered model ID. Send `Content-Type: application/json` and `x-goog-api-key` as headers. A minimal test request can use this body:

```json
{
  "contents": [
    {
      "parts": [
        { "text": "Reply with exactly: API_TEST_OK" }
      ]
    }
  ],
  "generationConfig": {
    "maxOutputTokens": 32
  }
}
```

Parse the real JSON response, safely extract candidate text, inspect any finish reason and token usage when available, and preserve sanitized raw JSON for inspection. Do not consider the returned phrase alone proof of success: verify HTTP status and response structure too.

### 8.4 Full health check sequence

Implement these checks as distinct results, not one vague green indicator:

1. **Endpoint/API reachability:** the server can connect to the configured Google API endpoint.
2. **Credential acceptance:** a real authenticated API operation succeeds or returns an informative rejection.
3. **Model availability:** the selected model is accessible and supports the chosen method where discoverable.
4. **Text generation:** a minimal real `generateContent` request succeeds and returns a valid response.
5. **Response parsing and timing:** report parse result, request latency, token usage, and finish reason if supplied.

Results must use `Passed`, `Failed`, `Warning`, `Not Tested`, or `Inconclusive`. Distinguish network reachability from accepted credentials, model permission, quota, and generation capability. Do not claim access to the user's Google account, billing, or quota beyond information actually returned by the API.

### 8.5 Results and report

Display an overall result plus per-check status, HTTP status, latency, generated text, sanitized error details, and one useful next troubleshooting step. Add tabs/sections for Summary, Generated Text, Request JSON, Response JSON, Error Details, and Timing. Implement Copy Text, Copy JSON, and Download Diagnostic Report. Redact API keys and sensitive headers from all views and exports.

### 8.6 Diagnostic guidance

Use actual HTTP status and provider error details before recommending a cause. Provide measured guidance for common situations:

- 400: inspect request schema, unsupported parameters, or invalid values.
- 401/403: inspect key validity, project/API access, and permissions.
- 404: verify model ID, endpoint path, and API version.
- 429: inspect quota/rate limits and retry policy.
- 5xx: report upstream service failure; suggest a later retry when appropriate.
- Network/TLS/DNS error: report the observed connectivity problem.
- Timeout: explain that the request did not finish within the configured timeout.

These are diagnostic hints, not absolute conclusions. Preserve the sanitized upstream error message when safe.

## 9. General API Health Check

Build a provider-aware health checker for all configured formats. The user can select a profile, validate endpoint configuration, execute a minimal authenticated request, test a model, measure latency, inspect errors, compare recent test results, and run sequential checks across selected profiles.

Use a request appropriate to the provider. Do not rely on a generic `HEAD` request as proof of authentication or successful generation. Report connectivity, authentication, model access, and generation as separate concepts. Avoid parallel bursts likely to trigger rate limits; offer sequential testing by default.

## 10. Model Explorer

Show discovered models where the provider supports discovery. Include provider filters, search, model ID/name, supported capabilities, context/token limits when returned, and a Test Model action. Do not invent model limits, prices, or capabilities. For providers without discovery, allow manual model IDs and label them as manually configured.

## 11. Response Inspector

Show:

- Parsed assistant/generated text
- Pretty-printed JSON and raw response when safe
- HTTP status and normalized result
- Request duration and time-to-first-token if genuinely measured
- Token usage and finish reason only when returned
- Safe response headers (remove cookies, credentials, and sensitive values)
- Provider error details
- Copy and download controls

If a response is non-JSON, render it safely as text. Streaming should be implemented only for adapters that truly support it; show chunks progressively and handle stream errors and cancellation. Never claim a metric that was not measured.

## 12. Request history

Use session memory or an explicitly enabled local history. Record only necessary safe metadata: timestamp, provider profile name, model ID, status, duration, success/failure, and sanitized error summary. Do not store keys, Authorization headers, or secret-bearing headers. Do not persist complete prompts/responses by default. Give users a clear way to disable and clear history.

## 13. Backend routes and modular adapters

Create validated, documented routes similar to:

- `GET /api/health`
- `POST /api/providers/test`
- `POST /api/providers/models`
- `POST /api/requests/execute`
- `POST /api/health-check/run`
- `POST /api/google-ai-studio/models`
- `POST /api/google-ai-studio/test-key`
- `POST /api/google-ai-studio/generate`
- `POST /api/google-ai-studio/health-check`

Adapt methods if a cleaner design is needed, while preserving the behavior. Avoid duplicating the Gemini logic between routes; share a well-typed service. Create a provider-adapter interface for request building, authentication, response parsing, streaming capability, and normalized errors.

Example high-level structure (you may improve it):

```text
api-forge/
├── client/
│   ├── src/components/
│   ├── src/pages/
│   ├── src/services/
│   ├── src/hooks/
│   ├── src/types/
│   └── src/styles/
├── server/
│   └── src/
│       ├── routes/
│       ├── providers/
│       ├── middleware/
│       ├── services/
│       ├── schemas/
│       └── utils/
├── tests/
├── .env.example
├── .gitignore
├── package.json
└── README.md
```

## 14. Security requirements — mandatory

- All upstream API requests and secrets must be handled by the backend; no direct provider-key calls from the browser.
- Accept a key only for the current request unless the user deliberately re-enters it. Do not persist secret values to disk, a database, localStorage, sessionStorage, IndexedDB, logs, browser history, or exports.
- Do not put credentials in URLs or query parameters.
- Avoid logging request bodies, complete headers, prompts, generated text, or raw upstream responses by default.
- Redact API keys, Authorization values, and known secret fields from errors, UI, exports, and tests.
- Use runtime validation, request-size limits, reasonable timeouts, rate limiting, security headers, and strict CORS configuration.
- Do not implement an unrestricted public proxy. In production, require appropriate access controls/rate limits before exposing request-execution routes to multiple users.
- **Custom URL SSRF protection is essential:** allow only HTTP/HTTPS as appropriate; resolve and reject loopback, private, link-local, cloud-metadata, and other forbidden destinations; validate IPv4 and IPv6; re-check redirect targets or disable redirects; prevent DNS rebinding where feasible. Prefer an allowlist for hosted deployments. Never let an external user use the server to reach internal services.
- Localhost/private development endpoints may be enabled only by an explicit development-only setting, never by a production bypass.
- Never reveal stack traces or secret values in production errors.
- Clear transient credential state after use and when the user clears credentials/leaves the page where practical. Do not claim client-side clearing can erase copies previously captured by a malicious environment.
- Use HTTPS in deployment and explain that credentials must not be submitted over insecure public HTTP.

## 15. Settings and import/export

Settings: theme, default timeout, preferred format, supported default generation options, response display, and history behavior. Import/export only non-secret configuration. Strip all API keys, Authorization values, and other secret-like fields automatically; validate imported data before use; never execute imported code.

## 16. Error model and reliability

Use a consistent safe error structure containing, where appropriate: safe error code, status, sanitized message, provider, duration, and retryability. Support schema errors, provider errors, invalid responses, timeouts, cancelled requests, and network errors. Retry only suitable transient failures, with a small configurable retry count; do not retry most validation or authentication errors. Add frontend error boundaries and graceful empty/loading states.

## 17. Automated tests

Create tests for:

- Configuration validation and provider selection
- Request-body construction for each adapter
- Response parsing and error normalization
- Gemini model-list and generation endpoint construction
- Missing/invalid model handling
- API key redaction in UI/log/export helpers
- Import/export secret stripping
- Timeout/cancellation behavior
- SSRF protection (including loopback/private IPv4/IPv6 and redirect handling)
- Important backend route validation

Use mocked upstream responses for automated tests so no real keys are needed. Add a separate manual live-test guide using the user's own API key. Do not report tests as passing unless you actually ran them.

## 18. Setup and deployment deliverables

Include:

- Complete frontend and backend files
- All required package manifests and scripts
- `.env.example` with placeholders only
- `.gitignore` excluding `.env`, secrets, build folders, and runtime logs
- Automated tests
- README containing prerequisites, install steps, development, build, start, testing, environment setup, live provider testing, and deployment instructions
- Example non-secret configuration profiles
- Troubleshooting instructions

Document simple commands, such as `npm install`, `npm run dev`, `npm run build`, `npm test`, and the correct production start command. Ensure scripts match the files actually created.

## 19. Definition of done

Do not declare the work complete until all applicable items are true:

- The frontend and backend build without errors.
- Main navigation and all advertised controls work.
- Provider profiles can be created/edited/deleted without saving secrets.
- At least the specified provider adapters make real requests through the backend.
- Google AI Studio has its own dedicated, functional health-check page.
- Gemini model discovery and text generation use real upstream responses.
- Success, failure, warning, and inconclusive states are not fabricated.
- JSON inspection, safe copy/export, and diagnostics work.
- No API keys appear in frontend bundles, storage, logs, history, or exports.
- Hosted custom-URL requests are protected against SSRF.
- Tests have been run; failures are fixed or clearly documented.
- README setup instructions reflect the actual implementation.

## 20. Instructions to the coding agent

1. Inspect the existing workspace and `task.md` before editing. Preserve useful existing work; do not overwrite unrelated files.
2. Implement the project, not only a plan, mockup, or sample snippet.
3. Work in logical stages, but continue through the stages without waiting for more user instructions unless a genuinely essential external credential or destructive choice is needed.
4. After each major stage, run the relevant checks. Fix errors instead of merely listing them.
5. Prefer complete, connected flows over numerous unfinished buttons. Any UI control shown as available must perform its advertised action or be explicitly marked unsupported/disabled with an explanation.
6. If a feature cannot be completed in the current environment, state exactly what is missing and finish all independent work possible.
7. At the end, report files created, implemented capabilities, commands to run, tests actually executed and their results, and any known limitations. Never claim unexecuted tests or unverified API success.
