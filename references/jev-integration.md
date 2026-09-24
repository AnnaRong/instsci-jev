# Jev Integration

This is an optional decision layer for InstSci workflows. The agent or
application remains responsible for deterministic checks, permissions, user
confirmation, browser actions, and the final report.

## Configuration

Before a live request, check only the environment for these values. Never
search shell history, repository files, browser storage, or unrelated files for
secrets.

```text
JEV_LANGUAGE      optional: en-US or zh-CN
JEV_API_KEY       required for a live Jev request
JEV_API_BASE_URL  optional: defaults to https://thejevai.com
JEV_MODEL         optional: defaults to typesafe/jev-1.13
```

If `JEV_API_KEY` is missing, stop before making a network request. Tell the user
to create a key in their Jev AI account and configure it locally; do not ask
them to paste the key into chat or place it in a skill file.

```bash
export JEV_API_KEY="sk_your_key_here"
export JEV_LANGUAGE="zh-CN"
export JEV_MODEL="typesafe/jev-1.13"
```

If the user asks for setup or help, report only whether the key is present,
never its value. Chinese guidance may be selected with `JEV_LANGUAGE=zh-CN`.

## Decision Types

Use one narrow question per decision:

- `choice`: select one item from a fixed `criteria` object;
- `score`: rate against an ordered `criteria` array;
- `noul`: estimate the probability that a yes/no statement is true.

Keep the state minimal. Remove API keys, cookies, credentials, unrelated
personal data, and unnecessary full-text paper content. Batch independent
questions only when they share the same state.

Useful InstSci examples:

- `choice`: select `open_access`, `publisher_browser`, `institution_login`,
  or `manual_review` as the next route;
- `score`: rate evidence quality from `0` to `3` using a defined rubric;
- `noul`: judge whether the current evidence is sufficient to ask for human
  review before another access attempt.

Never let a Jev result alone mark a publisher as supported, declare a PDF
download complete, or authorize an institutional action. Those outcomes still
require the InstSci and CloakBrowser evidence standard in `SKILL.md`.

## API Contract

The hosted endpoint is:

```http
POST ${JEV_API_BASE_URL:-https://thejevai.com}/v1/systemone
Authorization: Bearer $JEV_API_KEY
Content-Type: application/json
```

Example request:

```json
{
  "model": "typesafe/jev-1.13",
  "state": {
    "doi": "10.example/placeholder",
    "route_attempted": "open_access",
    "result": "no_verified_pdf"
  },
  "questions": {
    "next_route": {
      "type": "choice",
      "criteria": {
        "publisher_browser": "Use the visible publisher browser workflow",
        "institution_login": "Use the user's configured institution route",
        "manual_review": "Ask the user to review the access blocker"
      },
      "instructions": "Choose the safest next route using only the supplied state."
    }
  }
}
```

The expected successful response has `code: 0` and
`data.result.answers`. For `choice` and `score`, inspect the selected value,
probabilities, and confidence. For `noul`, inspect the `noul` value in the
range `0` to `1`. Treat all of these as advisory signals.

```bash
JEV_BASE_URL="${JEV_API_BASE_URL:-https://thejevai.com}"
curl -sS "$JEV_BASE_URL/v1/systemone" \
  -H "Authorization: Bearer $JEV_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "'"${JEV_MODEL:-typesafe/jev-1.13}"'",
    "state": {"route_attempted": "open_access", "result": "no_verified_pdf"},
    "questions": {
      "needs_review": {
        "type": "noul",
        "instructions": "Does this state require human review before another access attempt?"
      }
    }
  }'
```

Ask before making an otherwise unrequested live request because it can consume
credits or transmit paper metadata. Handle non-2xx responses, non-zero `code`,
missing answers, `401`, `402`, timeouts, and `429` explicitly. A failed Jev
request must not cause a consequential InstSci action to be repeated.

## Audit Boundary

When an audit trail is needed, record the model id, question definition,
redacted state summary, decision, confidence, and final action. Never record
the API key. The final action must remain attributable to deterministic
InstSci checks, user approval, or browser-backed evidence rather than to Jev
alone.
