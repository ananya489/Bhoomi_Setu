Bhoomi Setu – Land Record Digitization & Validation System



A full-stack land record digitization and validation platform developed as a team project.

Project Type: Team Project

Overview

Land records in many Indian jurisdictions still exist primarily as scanned paper documents, often in regional languages, distributed across state, district, and tehsil (sub-district) offices. Manually transcribing and cross-checking these records is slow and error-prone, and there is typically no reliable, tamper-evident history of how a record has changed over time.

Bhoomi Setu is a land record digitization and validation system that addresses this problem end to end. Officials upload scanned land-record documents through a web frontend; the backend processes them through an OCR pipeline built on the Mistral OCR API and extracts structured fields such as owner name, khasra number, khata number, village, tehsil, district, and state.

Extracted fields carry confidence scores. Documents that fall below a configurable confidence threshold are routed for manual review rather than being accepted automatically. Once validated, the extracted data can be used for land-record transactions such as ownership transfers and parcel subdivisions.

The system also provides role-based, jurisdiction-scoped access. State, district, and tehsil-level officials are restricted to their assigned jurisdiction, while verification officers are restricted to records assigned to them. These restrictions are enforced by the backend at the database-query level.

Tech Highlights

React + TypeScript + Vite frontend

FastAPI + Python backend

MongoDB with PyMongo

Mistral OCR for document processing

JWT authentication + bcrypt password hashing

Role-Based Access Control (RBAC)

Jurisdiction-scoped data access and anti-IDOR protections

Confidence-based OCR review workflow

SHA-256 hash-chained record history

Multi-language UI with i18next

Pytest-based automated testing

S3-compatible document storage with local fallback

Key Features

Authentication & Authorization

Signup and login with bcrypt-hashed passwords

JWT-based authentication with short-lived access tokens and longer-lived, individually revocable refresh tokens

Seven roles: super_admin, state_admin, district_admin, tehsil_officer, verification_officer, auditor, and citizen

Explicit permissions including UPLOAD_DOCUMENT, PROCESS_DOCUMENT, VIEW_RECORD, EDIT_RECORD, VERIFY_RECORD, APPROVE_RECORD, EXPORT_DATA, VIEW_AUDIT, and MANAGE_USERS

Jurisdiction-based access control across state, district, and tehsil levels

Verification officers are scoped to records explicitly assigned to them

Backend-enforced single-record access checks help prevent IDOR-style access through guessed IDs

Land Record Management

Scoped land-record listing and search

Record editing and approval workflows

Public land-record search and lookup by public record number

Permission-gated access to record operations

Document Processing & OCR

Document upload with file-type and size validation

OCR through a dedicated Python pipeline using the Mistral OCR API

Structured extraction of owner name, relative/father-husband name, village, tehsil, district, state, tauzi number, khata number, and khasra number

Hindi and English label matching

Per-field and overall confidence scoring

Configurable accept/review thresholds

Automatic Hindi/English document language detection

Manual review queues for low-confidence documents

Transactions & Verification

Ownership-transfer/sale transactions

Parcel subdivision into child khasra numbers

Area-consistency validation before transaction commitment

Transaction history by khasra number, owner, or transaction ID

Combined ownership/provenance history for a khasra number

Verification-officer workflow for low-confidence records

Corrected-field submission and record rejection

Auditor access to audit information and compliance reports

Record Integrity & Audit

Application-level SHA-256 hash-chained history of master-record changes

Each change is linked to the previous history entry

Current records can be verified against their recorded hash chain to detect tampering

Multi-stage governance workflow for master-record deletion requests

Immutable audit-log retrieval for auditors

Note: Bhoomi Setu uses an application-level hash chain for record integrity. It is not a distributed or consensus-based blockchain network.

Administration & Localization

Role-scoped user administration

State-level and district-level analytics

Officer and verification workflow management

Multi-language frontend using i18next/react-i18next

Interface translations for English, Hindi, Bengali, Gujarati, Kannada, Malayalam, Marathi, Odia, Tamil, Telugu, and Urdu

Why Bhoomi Setu?

The project goes beyond a conventional CRUD application by combining:

OCR-driven document processing

Structured information extraction

Confidence-based human review

Backend-enforced RBAC

Jurisdiction-aware data isolation

Land-record transaction validation

Tamper-evident record history

Multi-language frontend support

Automated security and workflow testing

System Architecture

flowchart TD
    User[Citizen / Officer / Admin]
    FE[React + TypeScript + Vite]
    API[FastAPI REST API]
    AUTH[JWT Authentication + bcrypt]
    RBAC[RBAC + Jurisdiction Scoping]
    SVC[Service Layer]
    DB[(MongoDB)]
    STORE[S3-compatible / Local Storage]
    OCR[Land Record OCR Pipeline]
    MISTRAL[Mistral OCR API]
    CHAIN[SHA-256 Hash-Chained History]

    User --> FE
    FE -->|REST calls| API
    API --> AUTH
    AUTH --> RBAC
    RBAC --> SVC
    SVC --> DB
    SVC --> STORE
    SVC -->|Document processing| OCR
    OCR --> MISTRAL
    OCR -->|Structured fields| SVC
    SVC --> CHAIN
    CHAIN --> DB

Architecture Overview

Frontend: React 19 + TypeScript single-page application built with Vite, with role-aware routing and typed API service modules.

Backend: FastAPI application exposing a versioned REST API under /api/v1, with role-specific routers and shared authentication, authorization, document, record, validation, and transaction modules.

Authorization: Permission checks and jurisdiction scoping are centralized and reused across backend routes rather than being implemented independently in each endpoint.

Database: MongoDB accessed through pymongo, with indexes configured for users, documents, roles/permissions, refresh tokens, processing jobs, audit logs, transactions, and master land records.

Document Storage: S3-compatible object storage through boto3, with a local filesystem fallback for development.

OCR Pipeline: A standalone Python package that validates documents, calls Mistral OCR, normalizes the response, detects language, extracts canonical land-record fields, and produces confidence information.

End-to-End Workflow

A user authenticates through the frontend.

The backend issues an access token and a revocable refresh token.

An authorized user uploads a scanned land-record document.

The backend validates the file and stores it using S3-compatible storage or the local fallback.

A document record and processing job are created in MongoDB.

The OCR pipeline sends the document to the Mistral OCR API.

Raw OCR output is normalized into a consistent internal representation.

Canonical land-record fields are extracted using Hindi/English label matching.

Field-level confidence values are calculated and combined into an overall confidence value.

Based on configured thresholds, the document is accepted or marked for review.

Low-confidence documents enter the appropriate manual verification workflow.

Validated information can be used for ownership-transfer or subdivision transactions.

Master-record changes are added to the SHA-256 hash chain.

Access to records remains restricted according to role and jurisdiction.

OCR & Document Processing Pipeline

The OCR pipeline is implemented as a standalone package under:

Land_Record_Mistral_OCR_Final_fixed (2)/
└── Land_Record_Mistral_OCR_Final_fixed/
    └── Land_Record_Mistral_OCR_Final_build/
        ├── src/land_ocr/
        └── tests/

Module

Responsibility

validation.py

Validates input files, extensions, size, existence, and non-empty content

mistral_client.py

Handles Mistral OCR API file upload and OCR requests

pipeline.py

Orchestrates validation, OCR, normalization, extraction, language detection, hashing, and result assembly

normalizer.py

Converts raw OCR blocks/tables/pages into a consistent internal structure

extraction.py

Maps OCR output to canonical land-record fields using Hindi/English labels

config.py

Reads OCR configuration and environment variables

cli.py / __main__.py

Provides command-line execution

api.py

Provides a thin standalone interface around the pipeline

Supported land-record fields include owner name, relative/father-husband name, village, tehsil, district, state, tauzi number, khata number, and khasra number.

When a Mistral API key is not configured, the repository supports development using pre-processed sample OCR output where applicable.

Authentication & RBAC

Authentication

Passwords are hashed using bcrypt.

Login issues JWT access and refresh tokens.

Refresh tokens are tracked server-side using a unique jti, allowing independent revocation.

Requests use Bearer-token authentication.

Authentication dependencies validate token type and active-user status.

Roles

super_admin
state_admin
district_admin
tehsil_officer
verification_officer
auditor
citizen

Jurisdiction Scoping

State, district, and tehsil officials are restricted to their assigned jurisdiction. Verification officers are restricted to records explicitly assigned to them.

The backend combines the requested record ID with the user's scope filter when performing single-record lookups. This prevents users from accessing another jurisdiction's data simply by obtaining or guessing a valid record ID.

Backend Architecture

app/
├── main.py
├── api/
│   ├── deps.py
│   └── routes/
├── core/
│   ├── config.py
│   ├── database.py
│   ├── security.py
│   ├── permissions.py
│   └── scope.py
├── models/
├── schemas/
└── services/

Important Components

Component

Responsibility

core/permissions.py

Role-permission mapping and permission dependencies

core/scope.py

Jurisdiction filters and scoped single-record access

core/security.py

Password hashing and JWT operations

services/ocr_service.py

Backend integration with the OCR pipeline

services/transaction_service.py

Transactions, subdivisions, and ownership history

services/blockchain_service.py

Existing hash-chain record-integrity implementation

services/storage_service.py

S3-compatible storage and local fallback

Frontend Architecture

The frontend is a React 19 + TypeScript application built with Vite and Tailwind CSS, using React Router.

Pages

Pages are organized around major workflows:

Login / Signup

Dashboard

Documents: upload, OCR viewer, extraction viewer, processing status, review, validation

Records: search, details, public search

Verification: task queue, verification workspace, completed verifications

Administration

Analytics

Audit

Settings

Services

The frontend contains typed API service modules for backend domains such as:

auth.service.ts
document.service.ts
record.service.ts
transaction.service.ts
validation.service.ts
verification.service.ts
admin.service.ts
audit.service.ts
dashboard.service.ts
extraction.service.ts
processing.service.ts

Localization

Localization is implemented with i18next and react-i18next, with translation files under src/locales/.

Database & Storage

MongoDB

MongoDB is the primary database and is accessed through pymongo.

The backend configures indexes for collections including users, roles, permissions, documents, refresh_tokens, processing_jobs, audit_logs, transaction, and master_records.

Document Storage

Documents can be stored using an S3-compatible object-storage service through boto3.

For local development, the application can use a filesystem fallback under the configured local storage directory.

Transactions & Record History

Transactions

Land-record transactions support:

Ownership transfer/sale

Parcel subdivision

Parent-to-child khasra relationships

Area-consistency validation

Transaction lookup by ID, khasra number, and owner

Ownership/provenance history

Record Integrity

Master-record changes are represented using an application-level SHA-256 hash chain.

Each history entry is linked to the previous entry's hash. The current record can be checked against its recorded chain information to detect modifications.

This approach provides application-level tamper evidence without requiring a distributed blockchain network.

Deletion Governance

Master-record deletion is handled through a staged request and approval workflow rather than an unrestricted destructive delete.

Testing

Testing uses pytest.

Backend Tests

tests/
├── conftest.py
├── test_phase1_critical.py
├── test_rbac_scoping.py
├── test_signup.py
└── test_transactions.py

These tests cover areas such as authentication, role behavior, jurisdiction scoping, anti-IDOR checks, CORS configuration, signup behavior, and transaction validation.

OCR Pipeline Tests

Land_Record_Mistral_OCR_Final_build/
└── tests/
    ├── test_extraction.py
    ├── test_extraction_actual.py
    ├── test_extraction_land_record1.py
    ├── test_normalizer.py
    ├── test_validation.py
    └── test_mistral_sample.py

Run Backend Tests

pytest

Run OCR Tests

cd "Land_Record_Mistral_OCR_Final_fixed (2)/Land_Record_Mistral_OCR_Final_fixed/Land_Record_Mistral_OCR_Final_build"
pytest

Tech Stack

Layer

Technology

Purpose

Backend

FastAPI + Python

REST API and backend workflows

Database

MongoDB + pymongo

Primary data store

Authentication

JWT + bcrypt

Authentication and password security

Object Storage

boto3 + S3-compatible storage

Document storage

OCR

Mistral OCR API

Document OCR

Frontend

React 19 + TypeScript

User interface

Build Tool

Vite

Frontend development/build

Styling

Tailwind CSS

UI styling

Routing

React Router

Frontend navigation

Localization

i18next / react-i18next

Multi-language interface

Testing

Pytest

Automated backend/OCR testing

Setup & Installation

Prerequisites

Python 3.11+

Node.js and npm

MongoDB local instance or MongoDB Atlas

Mistral API key for live OCR

Clone

git clone https://github.com/ananya489/Bhoomi_Setu.git
cd Bhoomi_Setu

Backend

Create and activate a virtual environment:

python -m venv .venv

Windows PowerShell:

.venv\Scripts\Activate.ps1

Linux/macOS:

source .venv/bin/activate

Install dependencies:

pip install -r requirements.txt

Environment Variables

Copy .env.example to .env and configure the required values.

Typical configuration includes:

MONGODB_URI=your_mongodb_connection_string
MONGODB_DB_NAME=land_records

SECRET_KEY=your_secret_key
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

Do not commit real credentials or secrets.

Run Backend

python -m uvicorn app.main:app --reload --port 8000

Run Frontend

cd ilrdvs-frontend
npm install
npm run dev

Run Both on Windows

The repository includes start.bat, which launches the backend and frontend in separate terminal windows.

API Documentation

Once the backend is running, interactive Swagger documentation is available at:

http://127.0.0.1:8000/docs

API Overview

All API endpoints are served under the /api/v1 prefix.

Area

Purpose

Auth

Signup, login, token refresh, logout, current-user profile

Documents

Upload, listing, status/detail retrieval, OCR results, document operations

Records

Scoped listing, retrieval, editing, approval

Transactions

Ownership transfers, subdivisions, transaction/history queries

Validation

Validation results, field validation, officer decisions

Public Portal

Public record search and lookup

Tehsil Officer

Dashboard, upload, OCR review, approval workflows

Verification Officer

Low-confidence tasks, corrected-field submission, rejection

Administration

User/role management, dashboards, analytics

Auditor

Audit logs, integrity verification, compliance reports

Record Integrity

Hash-chain verification and history retrieval

Integrity Governance

Multi-stage master-record deletion requests

Representative Endpoints

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

The /blockchain/ path remains in the API because it reflects the repository's existing implementation naming. The underlying mechanism is the application's SHA-256 hash chain, not a distributed blockchain.

Engineering Highlights

Centralized authorization: permission checks and jurisdiction scoping are implemented centrally and reused across routes.

Backend-enforced data isolation: authorization is applied at the API/database layer rather than relying only on frontend visibility.

Anti-IDOR protection: scoped filters are applied to single-record lookups as well as collection queries.

Decoupled OCR pipeline: OCR processing is maintained as an independent package with its own test suite.

Confidence-driven review: low-confidence OCR results can be routed for human verification.

Tamper-evident history: master-record changes are linked through SHA-256 hashes.

Governed deletion: master-record deletion follows a staged approval workflow.

Security-focused testing: tests cover authentication, authorization, jurisdiction boundaries, invalid tokens, and related security behavior.

Design Decisions

Modular OCR Pipeline

OCR extraction is separated from API route handlers so that validation, normalization, extraction, and related logic can be tested and executed independently.

Backend-Enforced Authorization

The backend is treated as the security boundary. Frontend UI restrictions are not relied upon as the sole authorization mechanism.

Structured Field Extraction

Raw OCR output is transformed into canonical land-record fields so downstream validation, transaction, and review logic can operate on consistent field names.

Confidence-Based Review

OCR results include confidence information so uncertain documents can be routed for manual review instead of being treated as equally reliable.

Hash-Chained Record History

Record changes are linked using SHA-256 hashes to provide application-level tamper evidence without the operational complexity of a distributed blockchain network.

Team Project

Bhoomi Setu was developed as a collaborative team project.

The system combines frontend development, FastAPI backend services, authentication and authorization, OCR-based document processing, field extraction and validation, land-record transaction handling, record-integrity mechanisms, and automated testing across multiple contributors.

Individual contributor ownership is intentionally not listed here unless explicitly documented in the repository.

Future Work

GIS Integration

GIS is not currently implemented as a production backend feature. The existing cadastral map-related frontend functionality uses sample data/simulated API behavior.

Potential future capabilities include:

Interactive land-parcel maps

Real parcel boundary visualization

Geographic coordinate storage

Map-based record search

Spatial querying

GIS-capable database/geospatial indexes

Integration with verified cadastral survey data

Other Future Work

Public cloud deployment of the backend

CI/CD automation

Broader automated test coverage

Published test-coverage metrics

Labeled OCR evaluation dataset

Measured OCR extraction-accuracy metrics

Structured logging and monitoring

Backend-verified analytics for dashboard data

Project Structure

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
│
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
│
├── Land_Record_Mistral_OCR_Final_fixed (2)/
│   └── Land_Record_Mistral_OCR_Final_fixed/
│       └── Land_Record_Mistral_OCR_Final_build/
│           ├── src/land_ocr/
│           └── tests/
│
├── scripts/
├── tests/
├── .env.example
├── pytest.ini
├── requirements.txt
├── setup_and_run.bat
├── start.bat
└── test_upload.html

Security Notes

Never commit .env files or real secrets.

Never expose MISTRAL_API_KEY, SECRET_KEY, database credentials, or storage credentials in source control.

Configure secrets through environment variables.

Configure CORS with explicit allowed frontend origins.

Backend permission checks and jurisdiction scoping must remain enabled in deployment.

Frontend visibility should not be treated as the security boundary.

License

No LICENSE file is currently present in the repository.

If this project is intended for public reuse, add an appropriate license file to the repository.

