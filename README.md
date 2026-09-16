# Bhoomi Setu – Land Record Digitization & Validation System

> A full-stack land record digitization and validation platform developed as a **team project**.

**Project Type:** Team Project
## Overview

Land records are often maintained as scanned documents in different formats and regional languages. Manually digitizing, validating, and managing these records can be time-consuming and error-prone.

**Bhoomi Setu** is a full-stack land record digitization and validation platform designed to streamline document processing and structured land-record management.

The system allows authorized users to upload scanned land-record documents, extract structured information through an OCR pipeline integrated with the **Mistral OCR API**, normalize the extracted data, and validate the results before they proceed through the appropriate record workflow.

The platform also supports role-based access control, jurisdiction-based data scoping, land-record transactions, verification workflows, audit history, and an application-level hash-chained record history.

> **Bhoomi Setu was developed collaboratively as a team project.**

---

## Key Features

### Authentication & Authorization

- Email/password signup and login
- Bcrypt-based password hashing
- JWT-based authentication with access and refresh tokens
- Role-based access control (RBAC)
- Multiple roles:
  - `super_admin`
  - `state_admin`
  - `district_admin`
  - `tehsil_officer`
  - `verification_officer`
  - `auditor`
  - `citizen`
- Fine-grained permission checks at the API level
- Jurisdiction-based data scoping
- Backend-level authorization and scope enforcement

### Land Records

- Land record listing, search, and detail views
- Jurisdiction-scoped record access
- Record editing and approval workflows
- Public/citizen record search
- Validation status tracking

### Document Processing & OCR

- Scanned document upload
- File type and size validation
- OCR processing using the **Mistral OCR API**
- Dedicated land-record OCR pipeline
- Structured field extraction
- Hindi and English field-label support
- Extraction of fields including:
  - Owner name
  - Father/Husband/Relative name
  - Village
  - Tehsil
  - District
  - State
  - Tauzi number
  - Khata number
  - Khasra number
- Per-field confidence information
- Configurable confidence thresholds
- Manual review workflow for uncertain extraction results
- OCR processing and validation result views

### Transactions & Record History

- Land-record transaction workflows
- Ownership transfer/sale transactions
- Land-record subdivision
- Child khasra creation during subdivision
- Area validation
- Verification task management
- Completed verification tracking
- SHA-256 hash-based record history
- Tamper-evident application-level record ledger
- Audit trail for record changes

> The hash-chained ledger is an application-level integrity mechanism and is not a distributed blockchain or consensus network.

### Administration & Localization

- Role and user administration
- Analytics dashboard
- Multi-language interface
- Localization using `i18next` / `react-i18next`
- Support for multiple Indian languages including Hindi, Bengali, Gujarati, Kannada, Malayalam, Marathi, Odia, Tamil, Telugu, and Urdu

---

# System Architecture

```mermaid
flowchart TD

    User[User: Citizen / Officer / Admin]

    FE[React + TypeScript + Vite Frontend]

    API[FastAPI REST API]

    AUTH[Authentication<br/>JWT + bcrypt]

    RBAC[RBAC + Jurisdiction Scoping]

    SVC[Service Layer<br/>Documents / Records / Transactions / Validation]

    DB[(MongoDB)]

    STORE[Document Storage<br/>S3-compatible / Local Fallback]

    OCR[Land Record OCR Pipeline]

    MISTRAL[Mistral OCR API]

    CHAIN[Hash-Chained Record History]

    User --> FE
    FE -->|REST API| API
    API --> AUTH
    AUTH --> RBAC
    RBAC --> SVC
    SVC --> DB
    SVC --> STORE
    SVC -->|Document Processing| OCR
    OCR --> MISTRAL
    OCR -->|Extracted Fields| SVC
    SVC -->|Record Changes| CHAIN
    CHAIN --> DB
Architecture Overview
Frontend

The frontend is located in:

ilrdvs-frontend/

It is built with:

React 19
TypeScript
Vite
Tailwind CSS
React Router

The frontend provides interfaces for authentication, document processing, extraction review, validation, land-record search, verification, administration, analytics, and audit workflows.

Backend

The backend is located in:

app/

and is implemented using FastAPI.

The backend separates responsibilities across:

API routes
Authentication
Authorization
Permissions
Jurisdiction scoping
Database access
Pydantic schemas
Business services
Document processing
Transactions
Validation
Record history
Database

The application uses MongoDB through pymongo.

MongoDB stores application data including:

Users
Documents
Processing jobs
Transactions
Land records
Refresh tokens
Audit logs
Roles and permissions
Document Storage

The system supports S3-compatible object storage through boto3, with a local filesystem fallback for development.

End-to-End Workflow
User
  ↓
React Frontend
  ↓
FastAPI REST API
  ↓
Authentication
  ↓
RBAC + Jurisdiction Scope
  ↓
Document Upload
  ↓
Document Validation & Storage
  ↓
OCR Pipeline
  ↓
Mistral OCR API
  ↓
OCR Response Normalization
  ↓
Structured Field Extraction
  ↓
Confidence Evaluation
  ↓
Validation / Manual Review
  ↓
Land Record / Transaction Workflow
  ↓
Hash-Chained Record History
Workflow Steps
A user authenticates through the React frontend.
The FastAPI backend validates authentication and permissions.
An authorized user uploads a land-record document.
The backend validates and stores the document.
A processing job is created.
The document is sent to the OCR pipeline.
The OCR pipeline uses the Mistral OCR API for document processing.
OCR output is normalized into a consistent structure.
Land-record fields are extracted from the normalized output.
Confidence information is generated for extracted fields.
Low-confidence results can be flagged for manual review.
Validated information proceeds through record and transaction workflows.
Record changes are added to the hash-chained history.
Users access records according to their roles and jurisdiction.
OCR & Document Processing Pipeline

The OCR functionality is maintained as a dedicated Python package:

Land_Record_Mistral_OCR_Final_fixed/
└── src/
    └── land_ocr/
Pipeline Modules
Module	Responsibility
validation.py	Validates input documents before processing
mistral_client.py	Handles communication with the Mistral OCR API
pipeline.py	Orchestrates the OCR and extraction workflow
normalizer.py	Converts raw OCR responses into a consistent structure
extraction.py	Extracts canonical land-record fields
config.py	Loads pipeline configuration
cli.py	Provides command-line execution
api.py	Provides an API wrapper for the pipeline
Extracted Fields

The pipeline processes fields such as:

Owner name
Father/Husband/Relative name
Village
Tehsil
District
State
Tauzi number
Khata number
Khasra number

The extraction logic supports Hindi and English field-label variations.

Confidence & Review

Extracted fields are associated with confidence information.

Configurable thresholds can identify uncertain extraction results and route them for manual review.

This provides a human-review workflow for cases where automated extraction requires verification.

Authentication & RBAC
Authentication

The backend authentication system includes:

Bcrypt password hashing
JWT access tokens
JWT refresh tokens
Refresh-token tracking
Token validation
Active-account checks
Bearer-token authentication
Roles
super_admin
state_admin
district_admin
tehsil_officer
verification_officer
auditor
citizen

Permissions are defined for operations such as:

UPLOAD_DOCUMENT
VERIFY_RECORD
APPROVE_RECORD
EXPORT_DATA
MANAGE_USERS
Jurisdiction-Based Access

The system applies jurisdiction-based access rules.

For example:

State-level officials are scoped to their assigned state.
District-level officials are scoped to their assigned district.
Tehsil-level officials are scoped to their assigned tehsil.
Verification officers are scoped to records assigned to them.

These restrictions are enforced at the backend/query level rather than relying only on frontend visibility.

Backend Structure
app/
├── main.py
├── api/
│   ├── deps.py
│   └── routes/
│       ├── auth.py
│       ├── documents.py
│       ├── records.py
│       ├── transactions.py
│       ├── validation.py
│       ├── public_portal.py
│       ├── super_admin.py
│       ├── state_admin.py
│       ├── district_admin.py
│       ├── tehsil_officer.py
│       ├── verification_officer.py
│       └── auditor.py
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
Component	Responsibility
main.py	FastAPI application setup
deps.py	Authentication and authorization dependencies
auth.py	Signup, login, and token operations
documents.py	Document upload and processing
records.py	Land-record operations
transactions.py	Land-record transactions
validation.py	Validation workflows
public_portal.py	Public record search
security.py	Password hashing and JWT handling
permissions.py	Role and permission management
scope.py	Jurisdiction-based query scoping
database.py	MongoDB connection and indexes
services/	Application business logic
Frontend Structure
ilrdvs-frontend/
└── src/
    ├── pages/
    │   ├── Login/
    │   ├── Signup/
    │   ├── Dashboard/
    │   ├── Documents/
    │   ├── Records/
    │   ├── Verification/
    │   ├── Administration/
    │   ├── Analytics/
    │   ├── Audit/
    │   └── Settings/
    │
    ├── components/
    │   ├── auth/
    │   ├── cards/
    │   ├── documents/
    │   ├── layout/
    │   ├── tables/
    │   ├── ui/
    │   └── validation/
    │
    ├── services/
    ├── locales/
    ├── routes/
    └── types/

The frontend uses domain-specific service modules to communicate with the backend API for authentication, documents, extraction, records, transactions, validation, verification, administration, analytics, audit, and processing.

Testing

The project includes an automated test suite using Pytest.

Backend Tests

Backend tests cover:

Signup
Duplicate-email handling
Authentication
RBAC
Jurisdiction scoping
Verification-officer record assignment
Super-admin access
Transaction workflows
Area validation
Critical authentication paths

Important test files include:

tests/
├── test_phase1_critical.py
├── test_rbac_scoping.py
├── test_signup.py
└── test_transactions.py
OCR Pipeline Tests

The OCR pipeline tests cover:

Field extraction
Sample OCR responses
Normalization
Input validation
Mistral response structures
Hindi-language OCR content

Example:

tests/
├── test_extraction.py
├── test_extraction_actual.py
├── test_normalizer.py
├── test_validation.py
└── test_mistral_sample.py
Run Tests

Backend tests:

pytest

OCR pipeline tests:

cd "Land_Record_Mistral_OCR_Final_fixed (2)/Land_Record_Mistral_OCR_Final_fixed/Land_Record_Mistral_OCR_Final_build"
pytest
Tech Stack
Layer	Technology	Purpose
Backend	FastAPI	REST API and backend application
Language	Python 3.11	Backend and OCR pipeline
Database	MongoDB	Persistent application data
Database Driver	PyMongo	MongoDB integration
Authentication	JWT	API authentication
Password Security	bcrypt	Password hashing
Frontend	React 19	User interface
Frontend Language	TypeScript	Type-safe frontend development
Build Tool	Vite	Frontend development and build tooling
Styling	Tailwind CSS	UI styling
OCR / Document AI	Mistral OCR API	OCR and document extraction
Storage	S3-compatible / Local filesystem	Document storage
Storage SDK	boto3	Storage integration
Testing	Pytest	Automated testing
Charts	Recharts	Analytics visualization
Localization	i18next / react-i18next	Multi-language interface
Setup & Local Development
Prerequisites
Python 3.11
Node.js
npm
MongoDB or a configured MongoDB connection
Mistral API credentials for live OCR processing
Clone the Repository
git clone https://github.com/ananya489/Bhoomi_Setu.git
cd Bhoomi_Setu
Backend Setup

Create a virtual environment:

python -m venv .venv
Windows PowerShell
.venv\Scripts\Activate.ps1
Linux/macOS
source .venv/bin/activate

Install dependencies:

pip install -r requirements.txt
Environment Configuration

Create a .env file based on .env.example.

Example:

MONGODB_URI=your_mongodb_connection_string
SECRET_KEY=your_secret_key
MISTRAL_API_KEY=your_mistral_api_key

Configure additional storage and application settings according to .env.example.

Never commit .env files or real API keys to GitHub.

Run the Backend
uvicorn app.main:app --reload --port 8000

FastAPI documentation:

http://127.0.0.1:8000/docs
Run the Frontend

Open another terminal:

cd ilrdvs-frontend
npm install
npm run dev

The frontend runs using the Vite development server.

Windows Start Script

The repository also includes:

start.bat

which can be used to start the backend and frontend together during local development.

API Overview

The FastAPI backend provides REST APIs for:

Authentication
Document processing
Land records
Transactions
Validation
Public record search
Verification
Administration
Audit history

When the backend is running, interactive API documentation is available through:

http://127.0.0.1:8000/docs
Engineering Highlights

Bhoomi Setu demonstrates practical software-engineering concepts including:

Modular FastAPI backend architecture
REST API-based frontend/backend communication
Separation of routes, services, schemas, and models
JWT-based authentication
Role-based access control
Jurisdiction-aware data scoping
Dedicated document-processing pipeline
Mistral OCR integration
Structured field extraction
Data normalization
Confidence-based review workflows
Land-record transaction validation
Hash-chained record history
Automated backend and OCR testing
Multi-language frontend support
Design Decisions
Modular OCR Pipeline

OCR functionality is separated into a dedicated pipeline instead of placing all document-processing logic inside API routes.

This keeps document processing modular and independently testable.

Backend Authorization

Authorization and jurisdiction scoping are enforced at the backend level rather than relying only on frontend visibility.

Structured Extraction

OCR output is converted into canonical land-record fields, making extracted information easier to validate and use in downstream workflows.

Confidence-Based Review

Extraction results can be flagged for manual review using configurable confidence thresholds.

Hash-Chained Record History

Record changes are linked through SHA-256 hashes to create a tamper-evident application-level history of modifications.

Team Project

Bhoomi Setu was developed as a collaborative team project.

The project combines multiple areas of software development, including:

Frontend development
Backend/API development
Authentication and authorization
Document processing
OCR integration
Data extraction
Data validation
Transaction workflows
Automated testing

This README describes the overall system and does not assign individual contributions without verified contribution information.

Future Work
GIS Integration

Integrate a dedicated GIS (Geographic Information System) layer to connect digital land records with geographic and cadastral information.

Potential capabilities include:

Interactive land-parcel maps
Geographic visualization of land records
Parcel boundary visualization
Linking land records with geographic coordinates
Map-based record search
Cadastral data visualization
Spatial querying of land parcels
Integration with suitable GIS services or spatial databases
Backend Deployment

Deploy the FastAPI backend to a cloud environment so that the complete application can be accessed remotely.

CI/CD

Introduce GitHub Actions or another CI/CD platform to automatically run tests and validation checks when code changes are pushed.

Test Coverage Reporting

Add automated test coverage reporting and publish coverage results.

OCR Evaluation

Create a labeled evaluation dataset and measure field-level OCR/extraction performance using reproducible evaluation metrics.

Observability

Add structured logging, monitoring, and error tracking for production environments.

Project Structure
Bhoomi_Setu/
│
├── app/
│   ├── api/
│   ├── core/
│   ├── models/
│   ├── schemas/
│   └── services/
│
├── ilrdvs-frontend/
│   └── src/
│       ├── components/
│       ├── pages/
│       ├── routes/
│       ├── services/
│       ├── locales/
│       └── types/
│
├── Land_Record_Mistral_OCR_Final_fixed/
│   └── ...
│
├── tests/
├── scripts/
├── docs/
├── requirements.txt
├── pyproject.toml
├── pytest.ini
├── .env.example
├── .gitignore
├── start.bat
└── README.md
Security Notes
Never commit .env files.
Never expose API keys or database credentials.
Configure secrets through environment variables.
Use appropriate production secrets when deploying.
Keep backend authorization enabled even when frontend permission checks are present.
License

No license file is currently present in this repository.

If the project is intended to be distributed as open-source software, an appropriate license can be added later.
