# tryouts-cnab-forge
# cnab-forge

> Production-grade ingestor for Brazilian CNAB 240 positional bank files,
> built with **Hexagonal Architecture (Ports & Adapters)** and **event-driven design** on AWS.

[![CI](https://github.com/Chico-BMC/tryouts-cnab-forge/actions/workflows/ci.yml/badge.svg)](https://github.com/Chico-BMC/tryouts-cnab-forge/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Java 21](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/projects/jdk/21/)
[![Maven](https://img.shields.io/badge/Maven-3.9%2B-blue.svg)](https://maven.apache.org/)

---

## 📌 Why this exists

Brazilian banks exchange billing data through **CNAB** (*Centro Nacional de Automação Bancária*) files — fixed-width positional files where every field is defined by character offset, not by name. These files are still the backbone of boleto (bank slip) processing, reconciliation and cash flow in most Brazilian companies.

Ingesting CNAB files at scale is deceptively hard:

- **Layouts vary per bank** (Itaú, Bradesco, BB, Santander) and per file type (240 vs 400, remessa vs retorno).
- **Files arrive in batches** over SFTP, often with hundreds of MB and thousands of records.
- **Downstream systems** (billing, reconciliation, notification) should not know about CNAB — they should consume **domain events**, not positional files.
- **Auditability** matters: every boleto must be traceable back to its source file, with idempotency guarantees on retries.

`cnab-forge` solves this by isolating the **CNAB parsing domain** from infrastructure (S3, DynamoDB, SNS) through **ports and adapters**, so layouts can evolve without touching the core, and infrastructure can be swapped without touching the domain.

---

## 🏛️ Architecture

### Hexagonal (Ports & Adapters) + Vertical Slices

The domain sits at the center and **knows nothing about AWS, Spring, or persistence**. It exposes **driving ports** (use cases) and requires **driven ports** (interfaces it needs fulfilled). Adapters implement those ports for the real world.

