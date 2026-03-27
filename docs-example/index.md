[Documentation Index](./index.md)

# Documentation Index — Orderflow

This index is the single source of truth for all documentation in the Orderflow repository. Every feature, module, workflow, and maintenance guide is linked here A-Z.

---

## Overview

Orderflow is a Node.js/Express REST API for order lifecycle management. It handles order creation, payment processing, fulfilment routing, and customer notification across multiple fulfilment partners.

- [Global AI Context](./aicontext.md) — project identity, terminology, writing rules, directory mapping, critical system behaviours

---

## module1-gettingStarted

Prerequisites, installation, environment configuration, and initial verification.

- [Getting Started Overview](./module1-gettingStarted/readme.md)
- [AI Context: Getting Started](./module1-gettingStarted/aicontext.md)

---

## module2-api

REST API reference — every endpoint, request shape, response shape, and error codes.

- [API Overview](./module2-api/readme.md)
- [AI Context: API](./module2-api/aicontext.md)
- [Orders Endpoints](./module2-api/ordersEndpoints.md)
- [Payments Endpoints](./module2-api/paymentsEndpoints.md)
- [Webhooks Endpoints](./module2-api/webhooksEndpoints.md)

---

## module3-services

Service layer documentation — business logic, orchestration patterns, and inter-service contracts.

- [Services Overview](./module3-services/readme.md)
- [AI Context: Services](./module3-services/aicontext.md)
- [Order Service](./module3-services/orderService.md)
- [Payment Service](./module3-services/paymentService.md)
- [Notification Service](./module3-services/notificationService.md)

---

## module4-dataModels

MongoDB schemas, field-level constraints, indexes, and data integrity rules.

- [Data Models Overview](./module4-dataModels/readme.md)
- [AI Context: Data Models](./module4-dataModels/aicontext.md)
- [Order Schema](./module4-dataModels/orderSchema.md)
- [Payment Schema](./module4-dataModels/paymentSchema.md)

---

## module5-workflows

End-to-end usage patterns combining multiple services and API layers.

- [Workflows Overview](./module5-workflows/readme.md)
- [AI Context: Workflows](./module5-workflows/aicontext.md)

---

## module6-docMaintenance

Documentation maintenance guide, sync rules, progress tracker, and changelog.

- [Documentation Maintenance Guide](./module6-docMaintenance/documentationMaintenanceGuide.md)
- [Documentation Progress Tracker](./module6-docMaintenance/documentationProgress.md)
- [Changelog](./module6-docMaintenance/changelog.md)
- [Changelog (JSON)](./module6-docMaintenance/changelog.json)
