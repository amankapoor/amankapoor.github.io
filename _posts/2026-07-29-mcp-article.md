---
title: MCP Article
date: 2026-07-29
published: false
layout: post
---

**The rule: a server can see nothing except the call it was given.** Not the
conversation, not the model, not the other servers, not the user. Every design
decision in the protocol follows from that isolation, and most confusion comes
from forgetting it.

It is why the spec's own framing is that "servers should be extremely easy to
build; host applications handle complex orchestration." A server is a function
with a description attached. The hard thinking lives in the host.

Two things people get wrong:

- **"The server can't use an LLM."** It can. It just cannot borrow the *host's*
  model. A server may call any model API with its own key, and the deprecation of
  Sampling (§8) explicitly pushes servers in that direction.
- **"MCP is an agent framework."** It is not. There is no loop, no planning, no
  memory. It carries capability, not intelligence. You can run a client and a
  server together with no model anywhere in the picture.

---

## 3. A call, end to end

Four steps. This is the entire runtime story.

```
   HOST                          CLIENT                        SERVER
    │                              │                             │
    │  1. what's out there?        │   server/discover           │
    │─────────────────────────────►│────────────────────────────►│
    │                              │◄────────────────────────────│
    │       protocol versions, capabilities, instructions, cache hints
    │                              │                             │
    │  2. what can it do?          │   tools/list                │
    │─────────────────────────────►│────────────────────────────►│
    │                              │◄────────────────────────────│
    │       name, description, input schema, output schema, annotations
    │                              │                             │
    │  ── the model reads the descriptions and decides ──         │
    │                              │                             │
    │  3. do it                    │   tools/call                │
    │─────────────────────────────►│────────────────────────────►│
    │                              │◄────────────────────────────│
    │       content blocks + structuredContent, or InputRequired  │
    │                              │                             │
    │  4. (only if InputRequired) answer and retry the same call  │
```

Step 2 is where nearly all of your leverage sits, and it is the step people
skip past. **The tool description is the interface.** It is the only thing
standing between the model and a wrong decision, and it is prose — no type
system checks it.

Measured on byld with 8 runs per variant: a vague tool description produced 3
tool calls out of 8, a concrete one produced 8 out of 8. Swapping the agent
framework changed nothing. The description was the entire lever.

---

## 4. INPUT — how a server says what it can do

Everything here is declared once and read by the model on every call, so it sits
in the prompt prefix and costs tokens forever. Write it like it matters.

| What you declare | What it does |
|---|---|
| `name` | what the model calls. Short, verb-like. This is what appears in its reasoning. |
| `title` | human-readable, for a tool picker in a GUI. Optional. |
| `description` | **the load-bearing one.** The whole docstring reaches the model. Say when to call it and when not to. |
| input schema | generated from your function signature and type hints. Required parameters get filled; optional ones get ignored. |
| `annotations` | `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`. Hints, not guarantees — a host may use them to auto-approve a read instead of prompting the user. |
| `icons`, `_meta` | display, and a slot for your own metadata such as a corpus version. |

Two practical findings:

**Required beats optional.** byld first made `authority` optional and the model
ignored it, stuffing the location into the query string instead. Made required,
it gets filled every time. If you need a value, do not make it optional and hope.

**Arguments the model should not see.** Some inputs belong to the harness, not the
model — a user id, a tenant, a session. Do not put them in the schema; inject them
host-side. In LangChain this is `InjectedToolArg`. The model cannot get wrong what
it never sees.

---

## 5. OUTPUT — what comes back

A tool result has two channels, and using only one is the most common mistake.

**Content blocks** — what the model reads. Text, images, audio, or a
`ResourceLink` pointing at something bigger instead of inlining it.

**`structuredContent`** — a JSON value validated against the output schema you
declared. This is for *code*, not the model. As of 2026-07-28 it can be any JSON
value, not just an object.

Why you want both: the model needs prose it can reason over, and your own code
needs data it can check. byld's grounding check has to answer "is this number one
we actually returned?" — that is a question about data, and parsing it back out of
the model's prose is a bad way to ask it.

A trap worth knowing, found the hard way. If you build the output schema from a
Python `TypedDict` with optional fields, the SDK serialises *every* field, so an
absent key goes out as an explicit `null` — while `NotRequired[str]` produces a
schema that rejects null. Every answer that omitted a field failed validation.
Declare optional fields as `NotRequired[str | None]`.

**Errors are content, not exceptions.** A tool that fails should return a result
saying what went wrong in words the model can act on. `is_error` marks it. An
exception ends the turn; a described failure lets the model recover or tell the
user something true.

**The abstention slot.** Research on citation fabrication found that a structured
output with no "unknown" option is what pushes models to invent one — the schema
leaves them nowhere to put "I don't know", so they fill the field. If your result
has no way to say *not found*, you have built a machine that fabricates. byld's
`status` field is that slot, and it is closed to four values.

---

## 6. IN BETWEEN — when one call is not enough

Sometimes a server cannot finish without something only the user has: a
confirmation, a missing parameter, an OAuth login.

Before 2026-07-28 the server would send a request *back* to the client mid-call.
That required a live session and a back-channel.

**2026-07-28 has no server-to-client requests at all.** Instead the server returns
`InputRequiredResult` — "I need these things" — and the client gathers them and
**retries the same call** with the answers attached, plus a sealed `request_state`
blob the server issued so it can resume without having remembered anything.

That is what makes the revision stateless, and it is the single biggest change.
The server keeps nothing between requests, so it can be restarted or replicated
under a running host without anyone noticing.

Two flavours: **form** elicitation for structured input, and **URL** elicitation
for things that must never pass through the model's context — credentials,
payment, OAuth. The second is genuinely useful and easy to miss.

One SDK gotcha: calling `ctx.elicit()` directly uses the *old* back-channel path
and fails on a stateless deployment. The 2026-07-28 way is a `Resolve(...)`
annotation on the parameter.

---

## 7. The full map, and who each part is for

The clearest way to hold this is by **who decides**:

| Primitive | Who decides to use it | Use it when |
|---|---|---|
| **Tools** | the **model** | something should happen, chosen by the model mid-conversation |
| **Resources** | the **application** | there is addressable content a host may want to attach as context |
| **Prompts** | the **user** | you want to offer a canned template the user picks explicitly, like a slash command |

Most servers need only tools. That is not a failure of ambition; it is the shape
of most capabilities.

The supporting cast:

- **Resource templates** — `corpus://{law}/{clause}`, URI patterns rather than a
  fixed list.
- **Completions** — autocomplete as the user types. Only for **prompt arguments
  and resource-template variables**, never for tool arguments. Worth knowing
  precisely, because it means a template variable can enumerate your inventory to
  anyone who asks.
- **Caching** — `ttlMs` and `cacheScope`, new in this revision. Applies to
  `server/discover`, `tools/list`, `prompts/list`, and all three resource reads.
  **Not to `tools/call`.** If your data is static and expensive, that asymmetry is
  the strongest argument for exposing it as a resource rather than only a tool.
- **Subscriptions** — one `subscriptions/listen` stream for change events.
  Designed to sit on an external pub/sub bus so it survives replicas.
- **Progress notifications** — for work slow enough that silence looks broken.
- **Pagination** — cursors. Note the high-level SDK server ignores them and
  returns everything; real paging means dropping to the low-level API.
- **Extensions** — a bundle of tools, resources and methods advertised as a unit,
  negotiated with the client. MCP Apps (server-rendered HTML in a sandboxed
  iframe) is one.

---

## 8. What this revision took away

More was removed than added, and this is the part most likely to waste your time
if you learn MCP from older material:

- **The `initialize` handshake is gone**, replaced by `server/discover`.
- **Sampling is deprecated** — a server borrowing the host's model. Integrate a
  model API directly instead.
- **Roots and logging are deprecated.**
- **Per-resource subscribe/unsubscribe is gone**, replaced by one stream.
- **Tasks moved out of core** into an extension, and the Python SDK ships the
  types without dispatching them. Long-running work is yours to build.

The lesson for a builder: **"use everything in the spec" is not a goal.** A
meaningful part of the spec is on its way out. Adopt what your capability needs.

---

## 9. What to actually build

For a server exposing a handful of deterministic tools — which is most servers:

**Do these. They are cheap and they pay.**
1. A tool description written like documentation for a new colleague. This is the
   highest-leverage text in the entire system.
2. An output schema, so your own code can check results as data.
3. Annotations. Four booleans that let a host skip a permission prompt.
4. `instructions` on the server, for what holds across every call. Cached.
5. Errors returned as content, with an explicit "unknown" state in the schema.

**Consider, when the shape fits.**
- **Resources**, if content is static, expensive, or large enough that inlining it
  on every call hurts. The caching asymmetry is the real argument. Return
  `ResourceLink`s *alongside* inline text, never instead — many hosts ignore
  resources entirely, and the tool result must stand alone.
- **Elicitation**, if a call genuinely cannot proceed without a human. Not as a
  substitute for the model asking a question, which is cheaper and keeps the
  conversation where it belongs.

**Skip until something forces you.** Prompts, sampling, tasks, subscriptions,
logging, pagination, apps.

---

## 10. The short version

MCP is a description format and a call convention. A server publishes what it can
do; a client asks; a host decides when. The protocol carries capability, and every
scrap of intelligence stays on the host's side of the line.

Which means the thing that determines whether your server is any good is almost
never the protocol. It is the quality of what sits behind the tool, and the
honesty of what the tool says about itself.

byld's own experience makes the point. Getting the protocol right took an
afternoon and worked first time. The tool behind it returned a real clause, a real
citation and a real computed number — for the wrong chapter of the law. No part of
MCP was ever going to catch that.

Get the capability right. The protocol is the easy half.