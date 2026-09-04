---
name: setup-even-terminal
description: Set up, verify, update, or reconnect Even Realities Even Terminal with Codex, including transport selection and retrieval of the active pairing URL and token.
---

# Setup Even Terminal

Set up the official `@evenrealities/even-terminal` bridge and finish with a verified connection the user can paste or scan in the Even app.

## Source of truth

Check current instructions before installing or updating:

- Even Realities: <https://www.evenrealities.com/terminal>
- npm package: <https://www.npmjs.com/package/@evenrealities/even-terminal>
- Codex CLI: <https://developers.openai.com/codex/cli>

Use current package metadata rather than caching a version in this skill.

## Workflow

1. Inspect prerequisites with `node --version`, `npm --version`, `codex --version`, and `codex login status`. Even Terminal requires Node.js 18 or newer. If Codex is not authenticated, run the official login flow and let the user complete interactive authentication.
2. Resolve existing state before changing it. Check the intended port, running `even-terminal` processes, and any system service. If a healthy service already matches the requested provider, project, and transport, reuse it. Preserve an unknown or mismatched listener and choose another free port unless the user authorizes replacing it.
3. Inspect the current npm release and lifecycle metadata, then install or update with `npm install -g @evenrealities/even-terminal@latest`. Verify `even-terminal --version` and inspect `even-terminal --help` because available flags may change.
4. Choose a reachable transport:
   - Prefer Tailscale when the host is connected and the phone is on the same tailnet. Confirm with `tailscale status` and `tailscale ip -4`, then use `--tailscale`.
   - Use the LAN address only when the phone and host share a reachable local network.
   - Treat `--expose` as a temporary public tunnel. Use it only when requested or when no private path works, and explain the exposure before opening it.
5. Start with `--provider codex`, the user's intended `--cwd`, a free port, and an auto-generated token unless persistence requires a stable token. Keep an ad-hoc process running. Create or modify a boot service only when persistent operation is requested or already part of the setup being repaired.
6. Capture the exact URL and full token printed by the active process. For an existing service, derive them from its authoritative configuration or startup log. The pairing URL uses the reachable host and includes at least `token` and `defaultProvider=codex`; preserve any configured client name.
7. Verify the active listener with an authenticated read such as `GET /api/info?provider=codex` using `Authorization: Bearer <token>`. Confirm the response identifies Codex before reporting success.
8. Return the full pasteable URL, token, host/transport, port, version, and process persistence. Tell the user to open Even Realities → Terminal Mode and scan the QR code or paste the URL manually.

## Credentials

Treat the pairing token as a secret because it authorizes access to the bridge. Reveal it only to the requesting user, never embed a live token in this skill or a repository, and avoid copying it into extra logs or scratch files. State that the token remains valid for the lifetime of an ad-hoc process or until a persistent service rotates it.
