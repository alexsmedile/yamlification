# Why We Should Adopt a Monorepo

Over the past eighteen months, our engineering organization has grown from
four services to nineteen. What started as a clean microservices separation
has become a coordination problem. Every cross-service change requires
opening pull requests in multiple repositories, synchronizing release timing,
and hoping that CI passes in the right order.

## The Coordination Tax

When service A depends on service B, and both need to change together, we pay
a coordination tax. Someone opens a PR in repo A, another in repo B, they
merge in the wrong order, something breaks in staging, and two engineers spend
half a day debugging an integration issue that would never have existed in a
monorepo.

I've watched this pattern repeat at least a dozen times this quarter. The cost
is not just the debugging time — it's the context switching, the cross-team
communication overhead, and the growing reluctance to make changes that touch
more than one service.

## Shared Tooling

In our current setup, each repository maintains its own linting config, CI
pipeline, test runner setup, and dependency pinning. When we update ESLint, we
update it nineteen times. When we want to enforce a new security rule, we open
nineteen PRs. This is obviously wasteful, but it's also risky — repos drift
apart, and enforcing consistency becomes a full-time job.

A monorepo solves this completely. One linting config. One CI pipeline. One
place to enforce standards. The tooling investment pays dividends across every
service simultaneously.

## Risks

I want to be honest about the downsides. A monorepo grows. Git history becomes
complex at scale. Build times increase unless you invest in incremental builds
and caching. Some teams find the loss of repo autonomy culturally uncomfortable.

None of these are blockers, but they are real costs that require investment in
tooling and process to manage well.

## Recommendation

Adopt the monorepo for all new services starting Q3. For existing services,
migrate gradually — start with the three services that change together most
frequently. Avoid a big-bang migration.
