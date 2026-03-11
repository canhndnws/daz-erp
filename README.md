# RNFP — National Vocational Training Platform

**Registre National de la Formation Professionnelle**
National Registry for Vocational Training, Skills, and Credentials — Democratic Republic of Congo

Built on the [Moqui Framework](https://www.moqui.org/) with PostgreSQL.

---

## Overview

RNFP is a national-scale platform that digitizes and governs the full lifecycle of vocational education:

- Register training institutions and accredit them per province
- Catalog national training programs by sector
- Maintain a national learner registry (INFP identifier system)
- Manage enrollments, examinations, and results
- Issue tamper-proof digital diplomas with hash-chain integrity
- Enable public credential verification via REST API or QR code

---

## Project Structure

```
rnfp-platform/                  ← Root Moqui component (daz-erp)
│
├── conf/
│   ├── MoquiDevConf.xml        Development config (PostgreSQL localhost)
│   └── MoquiProductionConf.xml Production config (PostgreSQL via env vars)
│
├── component.xml               Root component declaration
├── MoquiConf.xml               Module loader (loads all sub-modules in order)
│
└── modules/
    ├── rnfp-common/            Shared: centralized StatusItem registry + AuditLog
    ├── rnfp-government/        Ministerial document and regulatory reference registry
    ├── rnfp-organization/      Provinces, training sectors, centers, program authorizations
    ├── rnfp-program/           National training program catalog
    ├── rnfp-learner/           INFP national learner registry + enrollment management
    ├── rnfp-training-exam/     Standardized exam codes and examination results
    ├── rnfp-credential/        Digital diploma issuance, integrity chain, verification
    └── rnfp-api/               REST API layer (thin wrappers + Moqui REST screen)
```

### Module Dependency Graph

```
rnfp-common
    ├── rnfp-government
    ├── rnfp-organization
    │       └── rnfp-program
    │               └── rnfp-learner
    │                       └── rnfp-training-exam
    │                               └── rnfp-credential
    │                                       └── rnfp-api
    └── (all modules depend on rnfp-common)
```

---

## REST API

Base path: `/api/rnfp`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/learners` | Register a new learner (generates INFP ID) |
| `GET` | `/learners` | Search learners by name, ID, phone, email |
| `POST` | `/learners/{infpId}/enrollments` | Enroll a learner in an authorized program |
| `GET` | `/learners/{infpId}/credentials` | List credentials for a learner |
| `GET` | `/programs` | List training programs (filterable by sector) |
| `GET` | `/sectors` | List training sectors |
| `GET` | `/provinces` | List educational provinces |
| `GET` | `/centers` | List training centers |
| `POST` | `/credentials` | Issue a digital credential *(requires auth)* |
| `GET` | `/credentials/{id}/verify` | Verify credential authenticity *(public)* |

---

## Prerequisites

- Java 11+
- [Moqui Framework](https://github.com/moqui/moqui-framework) installed
- PostgreSQL 13+

---

## Setup

### 1. Database

```sql
CREATE DATABASE rnfp_dev;
CREATE USER rnfp_dev WITH PASSWORD 'rnfp_dev';
GRANT ALL PRIVILEGES ON DATABASE rnfp_dev TO rnfp_dev;
```

### 2. Run as a Moqui component

Link or copy this directory into your Moqui runtime:

```bash
# Linux / macOS
ln -s /path/to/rnfp-platform /path/to/moqui/runtime/component/daz-erp

# Windows (PowerShell, run as Administrator)
New-Item -ItemType SymbolicLink `
  -Path "C:\moqui\runtime\component\daz-erp" `
  -Target "E:\rnfp-platform"
```

### 3. Point Moqui to the dev config

```bash
java -jar moqui.war conf=component://daz-erp/conf/MoquiDevConf.xml
```

### 4. Override database connection (optional)

```bash
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=rnfp_dev
export DB_USER=rnfp_dev
export DB_PASS=rnfp_dev
```

---

## Production Deployment

Use `conf/MoquiProductionConf.xml`. Supply all secrets via environment variables — never commit credentials to source control.

| Variable | Description |
|----------|-------------|
| `DB_HOST` | PostgreSQL host |
| `DB_PORT` | PostgreSQL port (default: 5432) |
| `DB_NAME` | Database name |
| `DB_USER` | Database username |
| `DB_PASS` | Database password |
| `CRYPT_PASS` | Entity encryption passphrase (min 16 chars) |

```bash
java -jar moqui.war conf=component://daz-erp/conf/MoquiProductionConf.xml
```

---

## Key Design Decisions

- **Centralized status registry** — `rnfp.common.StatusItem` replaces per-module status enums; all statuses are grouped by `statusTypeId`
- **Credential integrity chain** — each credential issuance and revocation appends a SHA-256 hash record linking to the previous entry (append-only, tamper-evident)
- **INFP ID generation** — learner IDs follow the format `INFP-XXXXXX`, auto-sequenced and duplicate-protected
- **API layer isolation** — `rnfp-api` contains only thin REST wrappers; all business logic lives in domain modules
- **Full audit trail** — every mutating service call logs to `rnfp.common.AuditLog` with user, timestamp, entity, and before/after values
