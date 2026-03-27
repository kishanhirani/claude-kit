[Documentation Index](../index.md) / module1-gettingStarted

# Getting Started

This section covers everything you need to install, configure, and run Orderflow locally and in a staging environment.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Environment Configuration](#environment-configuration)
- [Database Setup](#database-setup)
- [Running Locally](#running-locally)
- [Verifying Setup](#verifying-setup)
- [Next Steps](#next-steps)

---

## Prerequisites

Before running Orderflow you need:

- **Node.js 20 LTS** (`node --version` should show `v20.x.x`)
- **npm 10+** or **pnpm 9+**
- **MongoDB 7** running locally or a connection string to a remote cluster
- **Stripe account** with a test-mode secret key
- **Git** installed and authenticated

---

## Installation

```bash
git clone https://github.com/your-org/orderflow
cd orderflow
npm install
```

---

## Environment Configuration

Copy the example environment file and populate each variable:

```bash
cp .env.example .env
```

**.env variables:**

```bash
# Server
PORT=3000
NODE_ENV=development

# MongoDB
MONGO_URI=mongodb://localhost:27017/orderflow

# Stripe
STRIPE_SECRET_KEY=[STRIPE_SECRET]
STRIPE_WEBHOOK_SECRET=[WEBHOOK_SECRET]

# JWT
JWT_SECRET=[REDACTED_API_KEY]
JWT_EXPIRES_IN=7d

# Notification (SMTP)
SMTP_HOST=smtp.mailtrap.io
SMTP_PORT=587
SMTP_USER=[SMTP_USER]
SMTP_PASS=[SMTP_PASS]
NOTIFICATION_FROM=no-reply@orderflow.internal
```

> [!WARNING]
> Never commit `.env` to version control. It is listed in `.gitignore`. Use your team's secrets manager for staging and production values.

---

## Database Setup

Orderflow uses MongoDB with Mongoose. No migrations are required — schemas are enforced at the application layer. Indexes are created automatically on first run via `mongoose.model()` definitions.

For local development, start MongoDB:

```bash
mongod --dbpath /usr/local/var/mongodb
```

Or use Docker:

```bash
docker run -d -p 27017:27017 --name orderflow-mongo mongo:7
```

---

## Running Locally

```bash
# Development mode with hot reload
npm run dev

# Production build
npm run build
npm start
```

The server starts on `http://localhost:3000` by default. You should see:

```
[2026-01-15 09:00:00] INFO: MongoDB connected { host: "localhost:27017", db: "orderflow" }
[2026-01-15 09:00:00] INFO: Server listening { port: 3000 }
```

---

## Verifying Setup

Run the test suite against a local MongoDB test database:

```bash
npm test
```

All tests connect to `mongodb://localhost:27017/orderflow-test`. The database is dropped and re-seeded before each test file.

To confirm the API is live, send a health check request:

```bash
curl http://localhost:3000/health
```

Expected response:

```json
{
  "status": "ok",
  "mongo": "connected",
  "uptime": 12.4
}
```

---

## Next Steps

- [API Overview](../module2-api/readme.md) — endpoint reference
- [Services Overview](../module3-services/readme.md) — business logic layer
- [Data Models Overview](../module4-dataModels/readme.md) — schemas and constraints
- [Workflows Overview](../module5-workflows/readme.md) — end-to-end usage patterns
