# Bhoomi Setu

### Land Record Digitization & Validation System

Bhoomi Setu is a full-stack platform for digitizing, reviewing,
validating, and managing land records. Authorized users upload scanned
documents; a dedicated OCR pipeline uses the Mistral OCR API to extract
structured fields, calculate confidence information, and route uncertain
results for manual review.

The platform combines OCR-based document processing with role-based
access control, jurisdiction-scoped data access, land-record transaction
workflows, and tamper-evident record history.

**Project type:** Team project\
**Repository:** https://github.com/ananya489/Bhoomi_Setu

------------------------------------------------------------------------

## Table of Contents

-   [Problem](#problem)
-   [Solution Overview](#solution-overview)
-   [Key Features](#key-features)
-   [Workflow](#workflow)
-   [Technology Stack](#technology-stack)
-   [Architecture](#architecture)
-   [Roles and Access Control](#roles-and-access-control)
-   [OCR Pipeline](#ocr-pipeline)
-   [Transactions and Record
    Integrity](#transactions-and-record-integrity)
-   [Localization](#localization)
-   [Project Structure](#project-structure)
-   [Getting Started](#getting-started)
-   [Run the Application](#run-the-application)
-   [API Documentation](#api-documentation)
-   [Testing](#testing)
-   [Security](#security)
-   [Limitations and Future Work](#limitations-and-future-work)
-   [Team](#team)
-   [License](#license)

## Problem

Land records in many Indian jurisdictions are maintained as scanned
paper documents and historical records distributed across state,
district, and tehsil offices. Manually reading, transcribing, and
cross-checking these documents can be time-consuming and error-prone.

Bhoomi Setu helps convert scanned land-record documents into structured
digital information, provides a review path for uncertain OCR results,
and maintains a traceable history of master-record changes.

## Solution Overview

1.  An authorized user uploads a land-record document.
2.  The backend validates the file and stores it.
3.  The OCR pipeline processes it using the Mistral OCR API.
4.  OCR output is normalized and mapped to standard land-record fields.
5.  Field-level and overall confidence information is calculated.
6.  Results below configured confidence thresholds are routed for manual
    review.
7.  Authorized officials review records and perform supported
    land-record operations.
8.  Master-record changes are added to a SHA-256 hash chain.

The system does not treat uncertain OCR output as automatically reliable
data.

## Key Features

### Document processing

-   File type, size, and content validation.
-   OCR through a dedicated Python pipeline using the Mistral OCR API.
-   Structured extraction of land-record fields.
-   Hindi and English label matching and language detection.
-   Field-level and overall confidence scoring.
-   Configurable confidence thresholds and manual review queues.

### Land-record management

-   Jurisdiction-scoped listing and search.
-   Record editing and approval workflows.
-   Public record lookup by public record number.
-   Permission-gated record operations.

### Verification and transactions

-   Verification queues for low-confidence records.
-   Corrected-field submission and record rejection.
-   Ownership-transfer/sale transactions.
-   Parcel subdivision into child Khasra numbers.
-   Area-consistency validation before transaction commitment.
-   Transaction and ownership/provenance history.

### Authentication and authorization

-   JWT authentication and bcrypt password hashing.
-   Short-lived access tokens and individually revocable refresh tokens.
-   Seven application roles and explicit permissions.
-   State, district, and tehsil jurisdiction scoping.
-   Verification-officer access limited to assigned records.
-   Backend-enforced single-record access checks to help prevent
    IDOR-style access.

### Record integrity and audit

-   Application-level SHA-256 hash-chained record history.
-   Each history entry links to the previous entry.
-   Record integrity verification against recorded hash-chain
    information.
-   Staged approval workflow for master-record deletion requests.
-   Auditor access to audit information and compliance reports.

> **Important:** Bhoomi Setu currently uses an application-level SHA-256
> hash chain. It is not a distributed, consensus-based blockchain
> network.

## Workflow

``` mermaid
flowchart TD
    A[User signs in] --> B[Authorized document upload]
    B --> C[File validation and storage]
    C --> D[OCR using Mistral OCR API]
    D --> E[Normalize OCR output]
    E --> F[Extract land-record fields]
    F --> G[Calculate confidence scores]
    G --> H{Meets configured threshold?}
    H -- Yes --> I[Continue through authorized record workflow]
    H -- No --> J[Manual verification queue]
    J --> K[Officer reviews, corrects, or rejects]
    K --> I
    I --> L[Supported record operations and validation]
    L --> M[Update master record]
    M --> N[Append SHA-256 hash-chain history]
    N --> O[Audit and integrity verification]
```

## Technology Stack

  Layer            Technology                     Purpose
  ---------------- ------------------------------ --------------------------------------
  Frontend         React 19, TypeScript           Web application
  Build tool       Vite                           Frontend development and build
  Styling          Tailwind CSS                   UI styling
  Routing          React Router                   Frontend navigation
  Backend          Python, FastAPI                REST API and workflows
  Database         MongoDB, PyMongo               Primary application database
  Authentication   JWT, bcrypt                    Authentication and password security
  OCR              Mistral OCR API                Document text recognition
  Object storage   boto3, S3-compatible storage   Document storage
  Local storage    Filesystem fallback            Local development storage
  Localization     i18next, react-i18next         Multilingual interface
  Testing          pytest                         Automated tests

## Architecture

``` mermaid
flowchart TD
    U[Citizen / Officer / Admin] --> FE[React + TypeScript + Vite]
    FE --> API[FastAPI REST API]
    API --> AUTH[JWT Authentication]
    AUTH --> ACCESS[RBAC + Jurisdiction Scoping]
    ACCESS --> SERVICE[Backend Service Layer]
    SERVICE --> DB[(MongoDB)]
    SERVICE --> STORAGE[S3-compatible / Local Storage]
    SERVICE --> OCR[Standalone OCR Pipeline]
    OCR --> MISTRAL[Mistral OCR API]
    OCR -->|Structured fields and confidence| SERVICE
    SERVICE --> HASH[SHA-256 Hash-Chained History]
    HASH --> DB
```

-   **Frontend:** React and TypeScript single-page application with
    role-aware routing and typed API service modules.
-   **Backend:** FastAPI application exposing a versioned REST API under
    `/api/v1`.
-   **Authorization:** Centralized permission checks and
    jurisdiction-scoping logic.
-   **Database:** MongoDB accessed through PyMongo.
-   **Document storage:** S3-compatible object storage through boto3,
    with a local filesystem fallback.
-   **OCR:** Standalone Python package for validation, OCR requests,
    normalization, extraction, language detection, and confidence
    information.
-   **Record integrity:** SHA-256 hash-chain history stored as
    application data.

## Roles and Access Control

The application defines seven roles:

  Role                     General purpose
  ------------------------ -------------------------------------------------------
  `super_admin`            System administration
  `state_admin`            State-level administration
  `district_admin`         District-level administration
  `tehsil_officer`         Tehsil-level land-record workflows
  `verification_officer`   Review of assigned verification tasks
  `auditor`                Audit and compliance access
  `citizen`                Citizen-facing access, including public record lookup

Explicit permissions include `UPLOAD_DOCUMENT`, `PROCESS_DOCUMENT`,
`VIEW_RECORD`, `EDIT_RECORD`, `VERIFY_RECORD`, `APPROVE_RECORD`,
`EXPORT_DATA`, `VIEW_AUDIT`, and `MANAGE_USERS`.

State, district, and tehsil officials are restricted to their assigned
jurisdiction. Verification officers are restricted to records assigned
to them. These checks are enforced by the backend, including for
individual record lookups. Frontend visibility is not treated as the
security boundary.

## OCR Pipeline

The OCR pipeline is maintained as a standalone Python package. It
validates input files, calls the Mistral OCR API, normalizes OCR output,
detects language, extracts canonical land-record fields, and assembles
confidence information.

Supported fields include: - Owner name - Relative / father / husband
name - Village, tehsil, district, and state - Tauzi number - Khata
number - Khasra number

### OCR package modules

  -----------------------------------------------------------------------
  Module                              Responsibility
  ----------------------------------- -----------------------------------
  `validation.py`                     Input-file validation

  `mistral_client.py`                 Mistral OCR API upload and requests

  `pipeline.py`                       Pipeline orchestration

  `normalizer.py`                     Normalization of OCR blocks,
                                      tables, and pages

  `extraction.py`                     Canonical field extraction using
                                      Hindi/English labels

  `config.py`                         OCR configuration and environment
                                      variables

  `cli.py` / `__main__.py`            Command-line execution

  `api.py`                            Standalone pipeline interface
  -----------------------------------------------------------------------

When a Mistral API key is not configured, the repository supports
development with pre-processed sample OCR output where applicable.

## Transactions and Record Integrity

Supported transaction workflows include: - Ownership transfer or sale. -
Parcel subdivision into child Khasra numbers. - Parent-to-child Khasra
relationships. - Area-consistency validation before transaction
commitment. - Transaction lookup by transaction ID, Khasra number, and
owner. - Ownership and provenance history for a Khasra number.

Master-record changes are represented using an application-level SHA-256
hash chain. Each history entry links to the previous entry's hash. The
system can compare the current record with recorded chain information to
help detect modifications.

Master-record deletion is handled through a staged request-and-approval
workflow rather than unrestricted destructive deletion.

**This is tamper-evident application-level record history, not a
decentralized blockchain or distributed consensus network.**

## Localization

The frontend uses `i18next` and `react-i18next`. Interface translations
are available for: - English - Hindi - Bengali - Gujarati - Kannada -
Malayalam - Marathi - Odia - Tamil - Telugu - Urdu

Translation files are maintained under `src/locales/` in the frontend.

## Project Structure

``` text
Bhoomi_Setu/
├── app/                                  # FastAPI backend
│   ├── main.py
│   ├── api/
│   │   ├── deps.py
│   │   └── routes/
│   ├── core/
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── security.py
│   │   ├── permissions.py
│   │   └── scope.py
│   ├── models/
│   ├── schemas/
│   └── services/
├── ilrdvs-frontend/                      # React + TypeScript frontend
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   ├── services/
│   │   ├── locales/
│   │   ├── routes/
│   │   └── types/
│   ├── package.json
│   └── vite.config.ts
├── Land_Record_Mistral_OCR_Final_fixed (2)/
│   └── Land_Record_Mistral_OCR_Final_fixed/
│       └── Land_Record_Mistral_OCR_Final_build/
│           ├── src/land_ocr/
│           └── tests/
├── scripts/
├── tests/
├── .env.example
├── pytest.ini
├── requirements.txt
├── setup_and_run.bat
├── start.bat
└── test_upload.html
```

## Getting Started

### Prerequisites

-   Python 3.11 or later
-   Node.js and npm
-   MongoDB local instance or MongoDB Atlas
-   Mistral API key for live OCR processing

### 1. Clone the repository

``` bash
git clone https://github.com/ananya489/Bhoomi_Setu.git
cd Bhoomi_Setu
```

### 2. Set up the backend environment

Create a virtual environment:

``` bash
python -m venv .venv
```

Activate it.

**Windows PowerShell:**

``` powershell
.venv\Scripts\Activate.ps1
```

**Linux / macOS:**

``` bash
source .venv/bin/activate
```

Install backend dependencies:

``` bash
pip install -r requirements.txt
```

### 3. Configure environment variables

Copy `.env.example` to `.env` and configure the values for your
environment.

Example:

``` env
MONGODB_URI=your_mongodb_connection_string
MONGODB_DB_NAME=land_records

SECRET_KEY=replace_with_a_secure_random_secret
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7

MISTRAL_API_KEY=your_mistral_api_key

S3_ENDPOINT_URL=your_s3_endpoint
S3_ACCESS_KEY=your_s3_access_key
S3_SECRET_KEY=your_s3_secret_key
S3_BUCKET_NAME=your_bucket_name
S3_REGION=your_region

USE_LOCAL_STORAGE_FALLBACK=True
LOCAL_STORAGE_DIR=./storage

ALLOWED_ORIGINS=http://localhost:5173
```

Use the configuration required by your environment. S3 settings are for
S3-compatible storage; local filesystem fallback is available for
development.

**Never commit `.env` files, API keys, database credentials, JWT
secrets, or storage credentials.**

### 4. Install frontend dependencies

``` bash
cd ilrdvs-frontend
npm install
```

## Run the Application

Open separate terminals from the repository root.

### Backend

Activate the virtual environment, then run:

``` bash
python -m uvicorn app.main:app --reload --port 8000
```

### Frontend

``` bash
cd ilrdvs-frontend
npm run dev
```

The Vite development server prints the local frontend URL in the
terminal.

### Windows shortcut

The repository includes `start.bat`, which launches the backend and
frontend in separate terminal windows. It also includes
`setup_and_run.bat` for setup assistance.

## API Documentation

The backend exposes a versioned REST API under `/api/v1`.

When the backend is running locally, interactive Swagger documentation
is available at:

http://127.0.0.1:8000/docs

### API areas

  -----------------------------------------------------------------------
  Area                                Purpose
  ----------------------------------- -----------------------------------
  Authentication                      Signup, login, token refresh,
                                      logout, current-user profile

  Documents                           Upload, listing, status/detail
                                      retrieval, OCR results, document
                                      operations

  Records                             Scoped listing, retrieval, editing,
                                      approval

  Transactions                        Ownership transfers, subdivisions,
                                      transaction/history queries

  Validation                          Validation results, field
                                      validation, officer decisions

  Public portal                       Public record search and lookup

  Tehsil Officer                      Dashboard, upload, OCR review,
                                      approval workflows

  Verification Officer                Low-confidence tasks,
                                      corrected-field submission,
                                      rejection

  Administration                      User/role management, dashboards,
                                      analytics

  Auditor                             Audit logs, integrity verification,
                                      compliance reports

  Record integrity                    Hash-chain verification and history
                                      retrieval

  Integrity governance                Multi-stage master-record deletion
                                      requests
  -----------------------------------------------------------------------

### Representative endpoints

``` http
POST /api/v1/auth/login
POST /api/v1/auth/signup
POST /api/v1/auth/refresh

POST /api/v1/documents
GET  /api/v1/documents/{id}

GET  /api/v1/records
PUT  /api/v1/records/{id}

POST /api/v1/transactions
GET  /api/v1/transactions/khasra/{khasra_number}/history

GET  /api/v1/blockchain/records/{record_id}/verify
GET  /api/v1/auditor/audit-logs
```

The `/blockchain/` route prefix reflects existing API naming. The
underlying implementation is a SHA-256 hash chain, not a distributed
blockchain.

## Testing

The project uses `pytest` for backend and OCR-pipeline tests.

### Backend tests

Run from the repository root:

``` bash
pytest
```

### OCR pipeline tests

``` bash
cd "Land_Record_Mistral_OCR_Final_fixed (2)/Land_Record_Mistral_OCR_Final_fixed/Land_Record_Mistral_OCR_Final_build"
pytest
```

The test suites cover authentication, role behavior, jurisdiction
scoping, anti-IDOR checks, CORS configuration, signup behavior,
transaction validation, OCR input validation, normalization, and field
extraction.

## Security

-   Never commit `.env` files or real credentials.
-   Keep `MISTRAL_API_KEY`, `SECRET_KEY`, database credentials, and
    storage credentials out of source control.
-   Configure CORS with explicit allowed frontend origins.
-   Keep backend permission checks and jurisdiction scoping enabled.
-   Do not rely on frontend visibility as the only access-control
    mechanism.
-   Review deployment configuration before exposing the application to
    public networks.

## Limitations and Future Work

### GIS integration

Production GIS integration is not currently implemented as a backend
feature. Existing cadastral-map-related frontend functionality uses
sample data or simulated API behavior.

Potential future capabilities: - Interactive land-parcel maps. - Real
parcel-boundary visualization. - Geographic coordinate storage. -
Map-based record search. - Spatial querying. - Geospatial indexes and
GIS-capable data services. - Integration with verified cadastral survey
data.

### Other planned improvements

-   Public cloud deployment of the backend.
-   CI/CD automation.
-   Broader automated test coverage and published coverage metrics.
-   A labeled OCR evaluation dataset.
-   Measured OCR extraction-accuracy metrics.
-   Structured logging and monitoring.
-   Backend-verified analytics for dashboard data.

## Team

Bhoomi Setu is a collaborative team project involving frontend
development, FastAPI backend services, authentication and authorization,
OCR-based document processing, field extraction and validation,
land-record transaction handling, record-integrity mechanisms, and
automated testing.

Individual contributor ownership is not listed because it is not
documented in the supplied project information.

## License

No `LICENSE` file is currently present in the repository. Until a
license is added, the project's reuse and redistribution terms are not
explicitly defined.

If the project is intended for public reuse, add an appropriate license
file.
