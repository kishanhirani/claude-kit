[Documentation Index](../index.md) / module1-gettingStarted / AI Context

# AI Context: Getting Started

This file provides localised context for AI assistants working on the Getting Started documentation section.

---

## 🏗 Section Identity

The **Getting Started** section covers installation, environment configuration, and initial verification for Orderflow. Its audience is a developer setting up the project for the first time — either local development or staging. It must be accurate, minimal, and link forward to deeper sections rather than duplicate their content.

---

## 📚 Key Terminology

- **`.env.example`** — the committed template for environment variables. Contains keys but no values.
- **`MONGO_URI`** — the MongoDB connection string. Local default is `mongodb://localhost:27017/orderflow`.
- **`STRIPE_SECRET_KEY`** — Stripe test-mode secret. Begins with `sk_test_` in development.
- **`STRIPE_WEBHOOK_SECRET`** — used to verify Stripe webhook signatures. Begins with `whsec_`.
- **`JWT_SECRET`** — used to sign and verify auth tokens. Must be at least 32 characters in production.

---

## 💾 Core File Affiliation

- `.env.example` — environment variable template
- `src/index.ts` — server entry point, MongoDB connection, Express app bootstrap
- `package.json` — `dev`, `build`, `start`, `test` scripts

---

## 🖇 Mandatory Cross-References

- `docs/module2-api/readme.md` — API reference (link after install instructions)
- `docs/index.md` — master index

---

## 🎨 Formatting Rules

- Use `bash` code blocks for all terminal commands
- Use `bash` code blocks (not `env`) for `.env` file examples
- Use `json` code blocks for API response examples
- Use `> [!WARNING]` for any security-sensitive configuration note

---

## 🚨 Strict Constraints

1. Never show real Stripe keys, JWT secrets, or SMTP credentials in examples — always use bracketed placeholders
2. Do not document internal Express middleware setup — document observable behaviour (what you run, what you see) only
3. Do not duplicate API endpoint details here — link to `docs/module2-api/` instead
