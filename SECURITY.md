# Security

## Reporting a vulnerability

Please email **liorp@memorial-project.org** with a description of the issue and
steps to reproduce. Do not open a public GitHub issue for security reports.

## Secret scanning

This repository uses [gitleaks](https://github.com/gitleaks/gitleaks) to block
secrets from being committed.

### Local (pre-commit)

One-time setup per developer machine:

```sh
brew install pre-commit   # or: pip install pre-commit
pre-commit install
```

After that, every `git commit` runs gitleaks on staged content.

### CI

`.github/workflows/gitleaks.yml` runs gitleaks on every push and pull request.

### If gitleaks blocks your commit

**Real finding (actual secret):**

1. Do **not** commit.
2. Rotate the secret at the provider immediately.
3. Move the value into a gitignored `.env` or a Firebase Functions secret
   (`firebase functions:secrets:set NAME`).
4. Reference the secret from code via `process.env` / `import.meta.env`.

**False positive (e.g. a value that only looks like a secret):**

1. Add an allowlist entry to `.gitleaks.toml` — by regex, path, or specific
   secret value.
2. Commit the updated `.gitleaks.toml` with a note explaining why the value
   is safe to check in.

## What NOT to commit

- `.env`, `.env.*` (use `.env.example` for templates)
- Firebase service account JSON files
- Google OAuth tokens, refresh tokens
- API keys for paid services (Anthropic, OpenAI, ElevenLabs, Stripe, etc.)
- Private keys (`*.pem`, `*.key`, SSH keys)

## Notes on Firebase Web API keys

Firebase Web API keys (the ones in `firebaseConfig` that ship to the browser)
are **safe to expose** — they identify the project, not authenticate it.
Security is enforced by:

1. Firebase Security Rules (Firestore, Storage, Realtime Database)
2. Google Cloud Console restrictions on the key itself:
   **HTTP referrers** + **API restrictions**

Make sure both are configured — see
<https://firebase.google.com/docs/projects/api-keys>.
