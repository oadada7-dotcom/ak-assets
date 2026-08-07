---
name: gpt-image-generation
description: Generate images through the locally installed OpenAI Codex CLI, using the user's authenticated ChatGPT session. Use whenever the user asks for a drawing, image, photo, sketch, illustration, render, mockup, or any other visual asset — including phrasings like "draw me", "make a picture of", "generate an image", "I need a graphic for". Do not use for editing an image the user already has, or for chart/diagram work that belongs in code.
---

# GPT Image Generation

Generate visual assets by shelling out to the local `codex` CLI, which is
authenticated against the user's ChatGPT subscription.

## Preflight — run this first, every time

```bash
command -v codex >/dev/null 2>&1 && codex --version
```

If this fails, **stop and tell the user** rather than attempting the
generation. Two common causes:

- **Not installed** — `npm install -g @openai/codex`
- **Not authenticated** — `codex login`

This skill also cannot run in a remote/cloud Claude Code session (Claude Code
on the web, GitHub Actions, or any container-backed environment). `codex` lives
on the user's own machine; a remote container has neither the binary nor the
ChatGPT session. If the preflight fails in such an environment, say so plainly
and offer the fallbacks below instead of retrying.

## Usage

Run the binary via Bash, substituting the user's request verbatim:

```bash
codex exec -- "Generate an image using the GPT Image engine with this exact prompt: [INSERT USER PROMPT HERE]"
```

Guidance on the substituted prompt:

- Pass the user's creative intent through faithfully. Expand a terse request
  ("a logo") into a usable description (subject, style, composition, palette,
  aspect) only when the added detail is clearly implied by the conversation —
  never invent brand, likeness, or content decisions on the user's behalf.
- Keep the prompt on a single line and escape any embedded double quotes.
- `codex exec` is non-interactive and can run long. Allow a generous timeout
  (300000 ms) and do not interrupt a run that is still producing output.

## Handling the result

`codex exec` is an agentic coding CLI, not a dedicated image endpoint. Read the
output before reporting back, and handle whichever of these actually happened:

- **An image file was written** — locate it (check the working directory and
  any path named in the output), confirm it exists, then surface it to the user
  with `SendUserFile`.
- **Codex wrote a script instead of an image** — say so. Do not describe a
  script as a generated image.
- **Codex reported it cannot generate images, or errored** — report the failure
  verbatim. Do not retry the same command more than once, and never fabricate a
  description of an image that was not produced.

Report what the tool actually returned. An empty or ambiguous result is a
failed generation, not a success.

## Fallbacks

If `codex` is unavailable and the session has other image tooling connected,
offer it rather than leaving the request unfulfilled — for example the
Higgsfield (`generate_image`), Pixelcut (`generate_media`), or Adobe Firefly
MCP tools, when present. Ask before switching providers: the user chose Codex
for a reason (it bills against their existing ChatGPT subscription), and another
service may consume separate credits.
