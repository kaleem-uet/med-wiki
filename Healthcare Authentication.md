# Healthcare Authentication (SMART on FHIR)

> SMART on FHIR is the **OAuth2-based authorization framework** for healthcare applications. If you're building an app that accesses FHIR APIs (especially Epic, Cerner, etc.), you'll implement SMART on FHIR.

> **Beginner primer:** If you've ever implemented "Sign in with Google" or "Login with GitHub," you've already done 90% of SMART on FHIR. It's OAuth 2.0 + OpenID Connect, with two healthcare-specific add-ons: (1) the EHR can hand your app a *patient context* at launch ("this token is valid for patient #12345 only"), and (2) the scopes look like `patient/Observation.read` instead of `repo:read`. The trickiest part isn't the auth flow — it's registering your app with each EHR vendor's developer portal, then getting each individual hospital to approve it.

## What Is SMART on FHIR?

**SMART** = Substitutable Medical Applications, Reusable Technologies

It's an **HL7 Implementation Guide** that defines how to:
1. Launch a healthcare app from within an EHR
2. Authenticate users
3. Authorize access to specific FHIR resources
4. Get an access token scoped to a patient/encounter

It builds on standard **OAuth 2.0** with healthcare-specific additions.

> **Analogy:** If you've ever built "Login with Google" or "Login with GitHub" — SMART on FHIR is the same concept but for EHRs. Instead of `accounts.google.com`, you redirect to `epic.example.com/auth`. Instead of getting a Google user profile, you get a FHIR patient context. The OAuth2 flow is nearly identical — just with healthcare-specific scopes and launch parameters.

---

## Two Main Launch Types

### 1. EHR Launch (App launched from inside the EHR)

> **Real-world example:** A doctor is reviewing a patient's chart in Epic. They click your app's icon in the sidebar. Epic passes your app the patient ID and encounter ID automatically — your app opens pre-loaded with that patient's context. The doctor never has to search or log in again. Think of it like a Slack app that opens with the current channel context.

```
Doctor is in Epic, clicks your app icon
    |
    v
Epic sends launch context to your app
(patient ID, encounter ID, practitioner ID)
    |
    v
Your app redirects to Epic's OAuth /authorize endpoint
    |
    v
Epic authenticates the user (already logged in)
    |
    v
Redirect back to your app with authorization code
    |
    v
Your app exchanges code for access token
    |
    v
Access token includes: patient ID, scopes, FHIR endpoint URL
    |
    v
Your app calls FHIR APIs with the token
```

**Key:** The EHR tells your app which patient/encounter is in context. You don't need to search.

### 2. Standalone Launch (App runs independently)

```
User opens your app directly (not from EHR)
    |
    v
Your app redirects to EHR's OAuth /authorize endpoint
    |
    v
User logs in (MyChart credentials for patients, EHR credentials for clinicians)
    |
    v
User selects a patient (if applicable)
    |
    v
Redirect back with authorization code
    |
    v
Exchange for access token
    |
    v
Call FHIR APIs
```

**Key:** User must authenticate and select context manually.

### 3. Backend Services (System-to-System)

> **Analogy:** This is exactly like **GitHub App authentication** — you register your app, get a private key, sign JWTs with it, and exchange them for access tokens. No human in the loop. Used for automated pipelines, cron jobs, and bulk data processing.

```
Your server creates a signed JWT (using your private key)
    |
    v
POST JWT to EHR's /token endpoint (client_credentials grant)
    |
    v
Receive access token (no user interaction)
    |
    v
Call FHIR APIs (typically for bulk data or system operations)
```

**Key:** No user involved. Uses asymmetric keys (RS384 or ES384). Used for backend integrations, bulk data export, etc.

---

## Scopes (What You Can Access)

> **Analogy:** SMART scopes are like **GitHub fine-grained personal access token permissions**. Instead of `contents:read` on a single repo, you say `patient/Observation.read` on a single patient. The `[context]/[resource].[permission]` pattern is just the healthcare version of `[scope]/[object]:[verb]`. If you can read a GitHub token's permission string, you can read a SMART scope.

SMART scopes follow a pattern. If you've used OAuth scopes (like `repo:read` in GitHub or `email profile` in Google), this is the same idea but more granular:

```
[context]/[resource].[permission]

patient/Patient.read         → Read patient's Patient resource
patient/Observation.read     → Read patient's observations
user/Patient.read            → Read any patient (clinician context)
system/Patient.read          → Backend system access to patients
launch                       → Request EHR launch context
launch/patient               → Request patient context in standalone launch
openid fhirUser              → Get user identity info
offline_access               → Get a refresh token
```

### Scope Examples
```
# Patient app (MyChart launch) — read patient's own data
scope: launch patient/Patient.read patient/Observation.read patient/Condition.read

# Clinician app (EHR launch) — read/write for the user's patients
scope: launch user/Patient.read user/Observation.write user/MedicationRequest.write

# Backend service — bulk data export
scope: system/Patient.read system/Observation.read
```

---

## SMART Discovery

> **Analogy:** SMART discovery is **the OAuth equivalent of `robots.txt` or `.well-known/openid-configuration`** — a predictable URL that tells your app where to send users for auth, which scopes are supported, and which launch flows work. You hit one endpoint and get a config blob you can hard-code against. Same pattern as OIDC; if you've consumed an OpenID discovery doc, you already know how to read this.

Every SMART-enabled FHIR server publishes its OAuth endpoints at a well-known URL:

```
GET [FHIR Base URL]/.well-known/smart-configuration

Response:
{
  "authorization_endpoint": "https://ehr.example.com/auth/authorize",
  "token_endpoint": "https://ehr.example.com/auth/token",
  "capabilities": ["launch-ehr", "launch-standalone", "client-public", "client-confidential-symmetric"],
  "scopes_supported": ["patient/Patient.read", "openid", "fhirUser", ...],
  ...
}
```

Also available via the FHIR CapabilityStatement:
```
GET [FHIR Base URL]/metadata
```

---

## Token Response

```json
{
  "access_token": "eyJ0eXAi...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "patient/Patient.read patient/Observation.read",
  "patient": "abc123",
  "encounter": "enc456",
  "id_token": "eyJ0eXAi..."
}
```

Note: `patient` and `encounter` in the token response tell your app the current context.

> **Analogy:** The `patient` and `encounter` fields in the token response are like **a JWT claim that scopes the token to a specific tenant**. Standard OAuth tokens grant access to *resources by type* (read messages, write files). SMART tokens grant access to *resources by tenant* — this specific patient, this specific visit. Always check these fields and use them to filter FHIR queries; never assume a token is valid for all patients.

---

## Implementation Tips

1. **Use a SMART client library** — Don't roll your own OAuth flow
   - JS: `fhirclient` (smart-on-fhir/client-js)
   - Python: `fhirclient`
   - .NET: `Hl7.Fhir.SmartAppLaunch`

2. **PKCE is required** — SMART on FHIR requires PKCE (Proof Key for Code Exchange) for public clients

3. **Handle token refresh** — Access tokens expire (typically 1 hour). Use refresh tokens for long-lived sessions.

4. **Test in sandboxes:**
   - SMART App Launcher (launch.smarthealthit.org) — test both launch types
   - Epic sandbox (fhir.epic.com)
   - Cerner sandbox (fhir.cerner.com)

5. **App registration is per-EHR-vendor** — Register separately with Epic, Cerner, etc.

6. **JWKS for backend services** — Host your public key at a JWKS URL. The EHR validates your JWT signature against it.

---

**See also:** [[FHIR Deep Dive]], [[Epic Systems]], [[HIPAA for Developers]]
