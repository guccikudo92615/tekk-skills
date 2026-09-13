---
name: secrets-and-crypto
description: 'Audits how credentials are stored, exposed and rotated — hardcoded keys, secrets in logs, error bodies, health endpoints and client bundles — plus the crypto underneath (password hashing, token randomness, hand-rolled schemes) and the supply-chain surface of the dependencies themselves. Use when reviewing config and env handling, logging, build/bundle output, auth token generation, or dependency changes.'
---

# Skill: secrets-and-crypto

**Skill:** the material that makes every other control work. A perfect
authorization layer is decorative if its signing key is in the bundle, its
tokens are guessable, or a postinstall script exfiltrates the environment. You
own where secrets live and how they leak; whether a *log line* is useful or
noisy is the Observability loop's lane.

## What to evaluate

1. **Hardcoded credentials.** API keys, tokens, passwords, private keys, webhook
   signing secrets or connection strings committed in source, config, test
   fixtures, seed data, CI workflow files or infrastructure manifests. A
   fallback default (`process.env.X || 'dev-secret'`) is the same bug wearing a
   disguise: in an environment where the var is unset, that constant becomes the
   production secret.
2. **Client-bundle exposure.** Only the framework's public prefix (`VITE_*`,
   `NEXT_PUBLIC_*`, …) may reach the browser. Look for a server secret imported
   into client code, a server-only module pulled into a shared bundle, or a key
   passed as a prop/initial state. Also check the reverse claim: a "public" key
   that is actually privileged (a service role, an admin API key).
3. **Leaks through output.** Secrets or PII in log lines, error responses, stack
   traces returned to the client, debug/health/status endpoints that echo
   configuration, redirect URLs carrying tokens in the query string, and
   third-party error/analytics payloads. When a value must be logged for
   support, log a *prefix* — a suffix identifies the key to anyone who has seen
   another one.
4. **Token and randomness quality.** Session ids, reset/verification tokens,
   API keys and nonces generated with a cryptographically secure source, of
   sufficient length, compared in constant time, and stored hashed rather than
   in plaintext. `Math.random`, a timestamp, a counter or a UUIDv1 for anything
   security-bearing is a finding.
5. **Password and key handling.** A modern password hash (argon2/bcrypt/scrypt)
   with sane parameters, never a bare digest; encryption using an authenticated
   mode with a per-message nonce, never ECB or a static IV; no hand-rolled
   scheme where a library primitive exists.
6. **Rotation and blast radius.** Whether a leaked value can be rotated at all:
   a secret duplicated across five services with no single source, a key
   embedded in a shipped client, a token with no expiry and no revocation path.
   Say what rotating actually costs today — that is often the finding.
7. **Supply chain.** Dependencies added or changed by this diff: install-time
   scripts, a package whose name is one character from a popular one, a git or
   URL dependency pinned to a moving ref, a lockfile change that does not match
   the manifest. Judge *this* change's risk — a general "run an audit tool"
   recommendation is not a finding.

## How to verify before you claim

- **Show the value and where it travels.** Quote the assignment (redacting the
  middle) and the line that ships, logs or returns it. A variable named
  `SECRET` in a server file is not, by itself, an exposure.
- **Check whether it is already dead.** A key that is committed but revoked, or
  a fallback that no deployed environment hits, changes the severity — say which
  you established and how. If it is live, rotation is part of the proposal, not
  an afterthought.
- **Cite the current guidance for crypto choices.** Parameter recommendations
  move; when the proposal turns on a specific cost factor or algorithm, check it
  rather than quoting a remembered number.
