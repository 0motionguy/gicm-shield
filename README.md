# @gicm/shield

Six security layers for AI agent APIs in one TypeScript package: auth, rate limiting, secrets, security headers, a prompt-injection guard and an audit log.

Shield packages the request-path checks an agent or LLM endpoint usually needs and wires them behind a single `Shield` class. It is for developers who expose an agent over HTTP and want working defaults built on Node's `crypto`, with zod as the only dependency. It is a set of building blocks, not an audited security product.

Part of [gICM](https://github.com/0motionguy/gICM), a family of OpenClaw agent packages. Full usage notes: [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md).

## What it does

- **Auth:** HS256 JWTs (secret of at least 32 characters), hashed API keys with scopes and expiry, in-memory sessions, PBKDF2-SHA512 password hashing, timing-safe comparisons, and scope checks with `*` and `action:*` wildcards.
- **Rate limiting:** token bucket, sliding window and fixed window limiters behind a `RateLimiter` facade, a multi-tier limiter where every tier must pass, and `X-RateLimit-*` response headers.
- **Secrets:** an environment backend with a prefix, an in-memory backend encrypted with AES-256, a composite fallback chain, key generators and a `redactSecrets` helper for logs.
- **Headers:** a CSP builder with four presets, CORS origin matching with wildcards, HSTS and frame options, and four ready-made header sets (`strictApi`, `webApp`, `development`, `publicApi`).
- **Input guard:** 18 built-in regex patterns for prompt-injection phrasing (instruction override, role injection, jailbreak terms, encoding hints, prompt-leak requests), a length limit, optional sanitising and custom patterns. It matches known phrasings; it is not a classifier.
- **Audit:** a ring buffer (10,000 events by default) with helpers for auth, rate-limit, secret-access and input-guard events, filtering, stats and an `audit` event stream.

## Example

```ts
import { Shield } from "@gicm/shield";

const shield = new Shield({ auth: { jwtSecret: process.env.JWT_SECRET! } });

const auth = shield.authenticate(req.headers.authorization ?? "");
if (!auth.authenticated) return res.status(401).end();
if (!shield.checkRate(auth.userId!).allowed) return res.status(429).end();

const guard = shield.checkInput(prompt, auth.userId);
if (!guard.safe) return res.status(400).json({ threats: guard.threats });
```

## How it's built

TypeScript (ESM) bundled with tsup and tested with vitest. Seven modules in `src/` are composed by `Shield`, an `EventEmitter`. Node's `crypto` supplies HMAC, PBKDF2, AES and random bytes.

## Repository layout

- `src/`: `auth`, `rate-limiter`, `secrets`, `headers`, `input-guard`, `audit`, `shield`, `types`
- `src/__tests__/`: vitest suite (35 test cases)
- `skill/SKILL.md`: the OpenClaw skill definition

## Run it locally

```bash
pnpm install
pnpm build   # tsup -> dist/
pnpm test    # vitest (add --run for a single pass)
```

## Status

Beta. Version 1.0.0, a single extracted release (February 2026) with a test suite and no external security review. The in-memory secrets backend keeps values in process memory only.
