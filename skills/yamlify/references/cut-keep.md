# What to Cut and Keep

## Always cut

- Transitional sentences ("In this section, we will discuss...")
- Restatements of the section header in the opening sentence
- Filler phrases ("It is important to note that...", "As mentioned above...")
- Examples that illustrate a point already captured in the value
- Redundant synonyms ("fast and performant", "clear and transparent")
- Authorial voice and hedging ("I think", "we believe", "it seems")
- Marketing framing and persuasive prose
- Badges, decorative headers, visual-only elements
- Fields whose only value is `TBD`, `unknown`, or equivalent — with no owner, deadline, or actionable next step attached. If the TBD has a deadline or owner, keep it.

## Always keep

- Every hard constraint, limit, or non-negotiable
- Every decision where the rationale is non-obvious — move rationale to a comment
- Every explicit uncertainty or open question (lost uncertainty is worse than lost facts)
- Every deadline, owner, and dependency
- The document's stated conclusion or summary — compress to one line under `summary:`, never drop silently
- Anything that would surprise a reader who only sees the YAML

**Decision heuristic:** would an agent acting on this YAML need to know it?
Yes → keep. No → cut.

## Comment usage

Comments are first-class — not afterthoughts. Use them to preserve what
doesn't fit cleanly into a key/value pair:

```yaml
# clarify key meaning
deadline: 2025-06-01  # hard cutoff — no extensions approved

# preserve dropped nuance
cache_ttl: 5m
# Short TTL prioritizes freshness; revisit if p99 latency degrades

# flag uncertainty
owner: engineering  # unclear if platform or product eng

# preserve decision rationale
auth: JWT  # evaluated OAuth2 and sessions; JWT chosen for statelessness
```
