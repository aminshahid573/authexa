# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Complete OAuth2 Authorization Server implementation in Go.
- Support for Authorization Code Flow (with PKCE), Client Credentials Flow, Refresh Token Flow, Device Authorization Flow, and JWT Bearer Token Flow.
- MongoDB integration for persistent storage of clients, users, and tokens.
- Redis integration for session management and rate limiting.
- Admin dashboard UI for managing OAuth2 clients and users.
- Prometheus metrics endpoint for monitoring.
- Docker Compose setup for easy local development and deployment.
- Initial project documentation (API, Architecture, Deployment, Flows).
- Open Source community files (Code of Conduct, Contributing guide, Security policy).
- OIDC ID Token support: `id_token` is now generated and included in token responses when the `openid` scope is requested (authorization code, refresh token, and device code grants). Added `IDTokenClaims` struct and `GenerateIDToken` to JWT utilities ([OpenID Connect Core 1.0 section 2](https://openid.net/specs/openid-connect-core-1_0.html#IDToken)).
- Refresh token introspection: the `/oauth2/introspect` endpoint now validates opaque refresh tokens by looking up their database signature when JWT verification fails, returning proper `active`/`inactive` status ([RFC 7662](https://www.rfc-editor.org/rfc/rfc7662)).

### Security
- HSTS header (`Strict-Transport-Security`) is now enabled in production with a 2-year max-age and `includeSubDomains` (RFC 6797).
- Session cookies now enforce `Secure=true`, `HttpOnly=true`, and `SameSite=Lax` in production (RFC 6265).
- Cookie-clearing operations (logout, invalid session) now include matching security attributes so browsers correctly delete the cookie.
- Added `Referrer-Policy: strict-origin-when-cross-origin` header to prevent leaking OAuth parameters in referrer URLs.
- Added `Content-Security-Policy` header with restrictive defaults suitable for an authorization server.
- CORS debug logging is now disabled in production to prevent leaking internal routing information (RFC 6454).
- CSRF middleware is now disabled in development and enabled in production, controlled by the `APP_ENV` environment variable.

### Changed
- Health check endpoint split into two separate endpoints for container orchestration:
  - `GET /health/live` returns 200 if the process is running (Kubernetes `livenessProbe`).
  - `GET /health/ready` checks MongoDB and Redis connectivity, returns 200 or 503 with per-component status (Kubernetes `readinessProbe`).
- Health check response format now follows the [draft-inadarei-api-health-check](https://inadarei.github.io/rfc-healthcheck/) convention with `status`, `components`, and `time` fields.
- Security headers middleware is now environment-aware, accepting `appEnv` to conditionally apply production-only headers.
- Renamed project from `oauth2-provider` to `authexa` across all import paths, configuration, and documentation.

### Fixed
- Device consent pages (`consent_device.html`, `device_success.html`) now use the public `base.html` layout instead of `admin.html`, fixing 500 errors caused by missing template variables in the device authorization consent flow.
- Layout rendering bug causing 500 errors in the device authorization consent flow.
