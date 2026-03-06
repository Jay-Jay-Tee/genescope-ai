# Security

GeneScope is a hackathon prototype. This document describes the current security posture, what is and isn't protected, and what should be addressed before any broader public deployment.

---

## Current State

### No API authentication

All REST endpoints — including `POST /api/predict` and `POST /api/ocr` — are unauthenticated. Anyone who can reach the deployed backend URL can submit requests. There is no API key, session token, or login requirement.

This is acceptable for a hackathon demo with no sensitive stored data. It is not acceptable for a production deployment handling real patient data.

### No data persistence

The platform does **not** store any user-submitted health data. Each prediction request is processed in-memory and the result returned in the same HTTP response. No health profiles, family histories, or risk results are written to a database.

This significantly reduces the data exposure risk for this prototype: there is no data to breach.

### HTTPS on Render

The deployed instance runs over HTTPS via Render's default TLS termination. All traffic between the browser and backend is encrypted in transit.

### Frontend auth token (non-functional)

`frontend/src/api/client.js` includes an interceptor that reads `auth_token` from `localStorage` and attaches it as a `Bearer` header. This is scaffolding for a future auth implementation — the backend does not currently validate this token.

### Console logging

The API client logs all request URLs and response status codes to the browser console. This is development logging and should be removed before handling real patient data.

---

## Data Sensitivity

The inputs collected by this platform are health-adjacent and potentially sensitive:

- Age, gender, weight, height
- Known medical conditions
- Family members' medical conditions and vitals
- Lifestyle data (smoking, diet, stress level)
- Uploaded medical documents (lab reports)

**None of this is currently stored.** However, the sensitivity of this data means any future persistent storage requires appropriate controls (encryption at rest, access controls, data minimization, and compliance with applicable health data regulations).

---

## Recommendations for Production

**Authentication.** Add user accounts with proper authentication before storing any data. JWTs with refresh tokens or a managed auth provider (Auth0, Supabase Auth) are reasonable starting points.

**API rate limiting.** The prediction endpoint runs the full AI pipeline per request with no rate limiting. Add rate limiting at the reverse proxy or application level to prevent abuse.

**Input validation.** FastAPI with Pydantic schemas validates request shape and types. Ensure validation is strict and all fields have appropriate max-length constraints — especially the code/text fields that feed into OCR.

**Uploaded document handling.** The OCR endpoint accepts file uploads. In production, validate file type strictly server-side (not just by extension), cap file size, and scan uploaded files before processing. Do not store uploaded files longer than the request lifecycle.

**Remove development logging.** Strip all `console.log` API request/response logging from `client.js` before handling real health data.

**HTTPS enforcement.** Already handled by Render for the demo. Ensure any self-hosted deployment terminates TLS and redirects HTTP to HTTPS.

**Data residency.** If deploying for users in the EU or handling data under GDPR or India's DPDP Act, ensure server location and data handling comply with applicable regulations.

---

## Reporting Issues

This is an open-source prototype. If you find a security issue, please open a GitHub issue describing the problem. For sensitive disclosures, contact the maintainers directly via GitHub.
