# instsci-jev

Personal Codex skill combining the upstream InstSci workflow with optional
Jev AI typed decisions.

## Install

Copy the `instsci-jev` folder into the Codex skills directory:

```bash
unzip -o instsci-jev.zip -d "${CODEX_HOME:-$HOME/.codex}/skills"
```

Then invoke it explicitly with:

```text
Use $instsci-jev for this task.
```

The skill does not contain an API key. Configure `JEV_API_KEY` locally only if
you explicitly want to make a live Jev decision request. InstSci PDF claims
still require its visible CloakBrowser evidence workflow.
