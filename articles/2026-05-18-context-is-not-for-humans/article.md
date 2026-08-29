---
title: "Stop writing agent context the way you write for humans"
type: essay
date: 2026-05-18
status: draft
---

# Stop writing agent context the way you write for humans

When I give an agent context, I usually hand it a Markdown file. A README, a spec,
a handoff note. It feels right — Markdown is what I'd give a teammate. Headings,
prose, a table or two. Easy to read.

But the agent isn't a teammate reading once. It's carrying that document through
every turn of a session, paying for it in tokens each time, parsing prose written
to be *pleasant* rather than to be *retrieved*. The thing that makes Markdown good
for me is exactly what makes it a poor fit for the agent: it's optimized for a human
reading top to bottom, once.

I think we should stop doing that. Context for an agent is machine input. It should
be written like machine input.

## The question I started from

I didn't begin with YAML. I began with a question: *what is the leanest, most
contextual, simplest format I can hand an agent?*

Three constraints, and they pull against each other.

- **Lean** — every token in the context window costs money and crowds out room to
  think. A 2,000-word README is mostly connective prose the agent doesn't need.
- **Contextual** — but I can't just compress it to a summary. Summaries throw away
  the specifics. The agent needs the actual endpoint names, the actual expiry
  values, the actual constraints — not "the API uses tokens."
- **Simple** — and whatever the format is, I have to be able to write it, read it,
  and trust it without special tooling. If it needs a parser to inspect, it's too
  much.

Most formats satisfy two of these and fail the third. Prose is contextual and
simple but not lean. A dense binary blob is lean but neither simple nor inspectable.
JSON is lean-ish and parseable but punishing to read by hand and has nowhere to put
nuance. I spent a while testing token counts across formats, on real documents, and
I kept landing in the same place.

## Why YAML, specifically

YAML satisfies all three, and one feature decides it: comments.

When you compress a document, the hard part isn't the facts — facts go into
key/value pairs cleanly. The hard part is the *nuance*. The aside. The "this is
true except when X." The reason a decision was made. In JSON that nuance has
nowhere to live, so it either gets dropped or smuggled into a string and lost. In
YAML it goes in a comment, and a comment is first-class: the agent reads it, a human
reading the file reads it, and it survives.

So the shape of an entry becomes: the key is a semantic label, the value is a dense
payload, and the comment carries the part that doesn't compress.

```yaml
auth:
  mechanism: JWT bearer token
  token_expires_after: 24h
  refresh_expires_after: 30d
  public_endpoints: [/health, /login]  # all others require Bearer token
```

That's the same authentication section you'd otherwise write as a paragraph. It is
not a summary — every fact is still there. It's a restructuring. Meaning stays;
format changes. And it's a fraction of the tokens, because the connective prose a
human needs to glide through a paragraph is gone, and the agent never needed it.

## Agents forget, and reminders are small

Here's the part people underrate: long sessions drift.

Early context decays. The agent that knew your project's conventions at turn 5 is
fuzzy on them at turn 80. The usual fix is to re-paste the whole document — which is
expensive, and ironic, because you're spending a large amount of context to correct
for context loss.

But a *reminder* doesn't need to be the whole document. It needs to be the spine.
A small, structured context file — what this company is, what this project is doing,
the three constraints that matter right now — fed back in at the right moment keeps
the agent on track for a tiny fraction of the cost. The leaner the format, the
cheaper the reminder, the more often you can afford to give one.

This is why "lean" isn't just a nice-to-have. It changes what's *possible*. A
60%-smaller context file isn't 60% cheaper to do the same thing — it's the
difference between a reminder you give once and a reminder you give every time the
session turns a corner.

## Retrieval and progressive disclosure

Two principles ended up doing most of the work, and they're older than any of this.

**Information retrieval** — context isn't a document to be read, it's a store to be
queried. The agent shouldn't scan a wall of prose to find the auth rule; it should
go to the `auth` key. Structure *is* the index. When context is shaped as labelled
data, retrieval is a lookup, not a search.

**Progressive disclosure** — you don't load everything at once. You load an overview
first, then drill into the part that's relevant. This is why I don't keep context in
one giant file. I keep multiple small YAML files — one per concern — and an index
that sits over them. The agent reads the index to orient, then pulls only the file
it needs. The overview is cheap; the detail is on demand.

This is the part that took the longest to arrive at, and it's the least flashy. The
win isn't a clever encoding. It's the boring discipline of: small files, one concern
each, an index on top, load what you need.

## What I'm not claiming

I'm not claiming Markdown is bad. It's the right format for humans, and the moment a
human needs to review or share the context, it should become Markdown again — round
-tripping back is part of the workflow, not an afterthought.

I'm not claiming this is summarization done well. It is explicitly *not*
summarization. The day you start dropping facts to save tokens, you've traded a
token problem for a correctness problem, and that's a worse trade.

And I'm not claiming you need a tool for any of this. You can write context as YAML
by hand today. The principles — machine-readable structure, comments for nuance,
small files, an index, progressive disclosure — are free.

## The actual claim

It's narrow. When you write context *for an agent*, write it for the agent.

The format you'd hand a colleague is optimized for a reader who goes top to bottom,
once, and enjoys the prose. An agent does none of those things. It queries, it
re-reads every turn, it pays per token, and it drifts over long sessions. Context
that respects those facts — structured, labelled, lean, with nuance preserved in
comments and detail behind an index — is a different artifact than a README, and it
should be.

I built [`yamlification`](https://github.com/alexsmedile/yamlification) because I
got tired of doing this conversion by hand. It turns Markdown into this kind of
YAML and back again. But the skill is just the convenient version of the argument —
the argument stands on its own.

```bash
/plugin marketplace add alexsmedile/yamlification
```
