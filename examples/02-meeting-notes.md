# Auth Service Planning — May 10 2025

Attendees: Marco (platform), Sara (security), Luca (mobile), Alex (product)

## Context

We've been running per-service session-based auth for two years. The mobile
team flagged that session storage is causing problems with their offline mode,
and the security team wants to move toward stateless tokens before the Q3
compliance audit. We agreed this is the right time to align on a new approach.

## Discussion

Marco opened by summarizing the two options on the table: JWT tokens vs
continuing with sessions but adding a shared session store. Sara pushed back
on the shared session store idea — she said it adds operational complexity
and creates a single point of failure. Luca agreed from the mobile side,
saying that JWT would simplify their offline token refresh flow significantly.

After some back and forth, we all agreed JWT is the right call. The stateless
nature is a hard requirement for mobile, and the security team is comfortable
with the token expiry approach.

For token TTL, Sara recommended 24 hours for access tokens based on industry
benchmarks. Refresh tokens at 30 days was raised by Luca and agreed by the
group. Marco noted that /health and /login must remain public — the load
balancer depends on /health and obviously /login can't require auth.

## Token Rotation

The one open question is refresh token rotation. Sara wants rotate-always
for maximum security. Luca is worried about the UX impact on mobile clients
— every refresh triggers a new token, which means more storage churn on
device. No decision reached. Sara will write up a short security brief by
next Friday (May 17).

## Actions

- Marco to start implementation of the refresh endpoint, targeting May 24.
- Sara to write security brief on rotation options by May 17.
- Alex to update the product spec to reflect JWT decision.

## Next Meeting

TBD — likely after Sara's brief is ready.
