# Secure Health

**Cybersecurity Framework for Secure Patient Data Transmission in Healthcare Networks**

An academic research prototype demonstrating core cybersecurity concepts for healthcare
data management: multi-factor authentication (mock OTP), role-based access control (RBAC),
private medical document uploads, OTP-protected document sharing, on-demand PDF report generation,
and fine-grained patient consent management. Built with Django 5.2, SQLite, and Bootstrap 5.

> [!WARNING]
> **PROTOTYPE DISCLAIMER**  
> All patient records, clinical measurements, medical histories, documents, and user accounts in this system are **entirely fictional**.  
> This system is strictly an academic research prototype. It is **not** suitable for real clinical use, HIPAA/GDPR compliance certification, production deployment, or any actual healthcare application.

---

## Technical Documentation Manual

For the complete technical architecture, sequence flows, ER diagram, threat model, testing breakdown, and maintenance guide, see the comprehensive manual:

📖 **[Technical Documentation Manual](docs/TECHNICAL_MANUAL.md)**

---

## Technology Stack

| Layer | Technology |
| :--- | :--- |
| **Backend** | Python 3.12+, Django 5.2 |
| **Database** | SQLite (local file `db.sqlite3` with WAL mode) |
| **Frontend** | Django Templates, Bootstrap 5.3.3, Bootstrap Icons 1.11.3 |
| **Authentication** | Email-driven mock OTP (SHA-256 hashed, session-based) |
| **PDF Reports** | ReportLab 4.2.5+ (in-memory streaming generation) |
| **Static Serving** | WhiteNoise 6.9 |

---

## Implemented Features

### Phase 1 — Project Foundation
- Django project structure with SQLite configuration
- Responsive Bootstrap 5 UI with navigation bar and custom healthcare CSS styling
- Public home landing page and clear prototype warning disclaimers

### Phase 2 — Authentication & Role-Based Access Control
- **Mock OTP Login** — email entry → 6-digit cryptographic code displayed in demo panel → session established
- Four user roles: **Admin**, **Doctor**, **Management**, **Patient**
- Role-specific dashboard views
- Patient directory and detail views with strict server-side RBAC enforcement
- Application audit logging for all authorization attempts

### Phase 3 — Medical Documents & Secure Sharing
- Private medical document uploads (PDF, JPG, PNG; max 5 MB) stored outside web root in `media_private/`
- Extension filtering and magic-byte header inspection (protecting against MIME spoofing)
- Authenticated streaming download endpoint (no direct public media URLs)
- **OTP-Protected Document Sharing** — doctor creates a grant for a registered recipient → 15-minute expiry → OTP challenge
- Centralized **Audit Log** interface for administrators with outcome filtering (`SUCCESS`, `DENIED`, `FAILED`, `INFO`)

### Final Phase — PDF Patient Reports & Consent Management
- **On-Demand PDF Patient Summaries** — generated in-memory via ReportLab without saving unencrypted files to disk; includes security watermark, patient demographics, allergies, medical history, and disclaimer
- **Patient Consent Management** — patients hold sovereign control to grant and revoke access to their medical records:
  - Purpose-specific grants: `view_record`, `access_documents`, `generate_report`
  - Dynamic authorization gating: assigned doctors require both assignment AND active patient consent to access documents or generate PDF reports
  - Immediate revocation: revoking consent instantly blocks future clinical access
  - Append-only event trail: `ConsentEvent` preserves a permanent, chronological decision log
  - Patient self-service UI and administrator audit view

---

## Excluded Features (Not Implemented)

The following are explicitly **not implemented** in this academic prototype:

- Live email or SMS telecom delivery (OTPs appear in on-screen demonstration panels)
- End-to-end asymmetric encryption for stored files
- Hardware-enforced tamper-proof audit logs or HSM integration
- HIPAA, GDPR, or NHS statutory compliance certification
- Production secret management (vaults / KMS)
- Biometric authentication (FIDO2 / WebAuthn)
- Break-glass or emergency override access workflows
- External cloud services, Docker containers, Celery, Redis, or PostgreSQL

---

## Installation and Quick Start

### 1. Prerequisites
- Python 3.12 or later
- `git`

### 2. Clone the repository
```bash
git clone https://github.com/Anurag7645/patient_data_security.git
cd patient_data_security
```

### 3. Create and activate virtual environment
```bash
python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
```

### 4. Install dependencies
```bash
pip install -r requirements.txt
```

### 5. Apply database migrations
```bash
python3 manage.py migrate
```

### 6. Seed fictional demo data and consents
```bash
python3 manage.py seed_demo
```
*Creates repeatable demonstration user accounts, patient records, demo documents, and sample consent grants. Safe to rerun.*

### 7. Collect static files
```bash
python3 manage.py collectstatic --noinput
```

### 8. Start the development server
```bash
python3 manage.py runserver
```
Navigate to **http://127.0.0.1:8000/** in your browser.

---

## Demo Accounts

All accounts authenticate via **passwordless mock OTP** (no real email sent; code is displayed on the verification screen).

| Email Address | Role | Name / Purpose | Pre-configured Data |
| :--- | :--- | :--- | :--- |
| `admin@securehealth.demo` | **Admin** | Alex Morgan | Full access; document vault; system audit log viewer |
| `doctor1@securehealth.demo` | **Doctor** | Dr. Sam Carter | Assigned to Chris Blake (SH-P-0001) & Taylor Ford (SH-P-0002) |
| `doctor2@securehealth.demo` | **Doctor** | Dr. Ria Patel | Assigned to Morgan Hayes (SH-P-0003) & Jamie Ward (SH-P-0004) |
| `management@securehealth.demo`| **Management**| Jordan Chen | Directory inspection; zero access to medical documents or reports |
| `patient1@securehealth.demo` | **Patient** | Chris Blake | Pre-granted consent for Doctor 1; has uploaded medical documents |
| `patient2@securehealth.demo` | **Patient** | Taylor Ford | No initial consent granted (use to test consent granting flow) |
| `patient3@securehealth.demo` | **Patient** | Morgan Hayes | Assigned to Doctor 2 |
| `patient4@securehealth.demo` | **Patient** | Jamie Ward | Assigned to Doctor 2 |

---

## Automated Tests

Run the full automated test suite:

```bash
python3 manage.py test core --verbosity=2
```

### Current Test Suite Status
```text
Found 97 test(s).
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.................................................................................................
----------------------------------------------------------------------
Ran 97 tests in ~1.4s

OK
Destroying test database for alias 'default'...
```

### Test Classes and Coverage (97 Tests Total)

| Test Class | Count | Covered Security Behaviors |
| :--- | :---: | :--- |
| `OTPModelTests` | 7 | Token generation, cryptographic verification, expiry, single-use, attempts |
| `LoginViewTests` | 4 | Email submission, unknown emails, demo panel rendering, session initialization |
| `VerifyViewTests` | 6 | Correct/incorrect/expired/reused OTPs, attempt limits |
| `DashboardRBACTests` | 5 | Role dashboard isolation for Admin, Doctor, Management, Patient, Anonymous |
| `PatientListRBACTests` | 5 | Directory access boundaries across all roles |
| `PatientDetailRBACTests` | 8 | Assigned doctor access, unassigned doctor denial, patient isolation, 404 |
| `DocumentUploadTests` | 7 | Allowed extensions, magic-byte forgery rejection, 5 MB file size limit, audit |
| `DocumentAccessAndDownloadTests` | 5 | Authenticated private streaming, assigned doctor access, management denial |
| `OTPProtectedSharingTests` | 5 | Sharing grant creation, recipient verification, 15-minute expiry, wrong user denial |
| `AuditLogTests` | 3 | Admin log review, non-admin denial, outcome filtering |
| `IntegrationAndStartupTests` | 3 | Clean imports, form validation, reverse URL pattern resolution |
| `PatientPDFReportTests` | 12 | PDF streaming, disclaimer watermark, demographics rendering, doctor assignment |
| `ConsentModelTests` | 4 | Grant string representations, active properties, purpose independence |
| `ConsentManagementViewTests` | 23 | Patient self-service grant/revoke, cross-patient isolation, doctor consent gating, admin access, append-only event trail, idempotency, audit logging |

---

## Manual Verification Checklist

- [x] **Demo Login:** Submit `admin@securehealth.demo`, retrieve 6-digit OTP from panel, successfully authenticate.
- [x] **RBAC Enforcement:** Log in as `management@securehealth.demo`, visit `/patients/SH-P-0001/` and attempt to download documents or generate PDF reports—observe 302 redirect and `DENIED` audit entry.
- [x] **Consent-Gated Document Access:** Log in as `doctor1@securehealth.demo`, visit `SH-P-0002` (Taylor Ford) and observe document access is blocked due to lack of consent.
- [x] **Consent Grant:** Log in as `patient2@securehealth.demo`, visit `/patients/SH-P-0002/consent/`, and grant `Access Patient Documents` to `doctor1@securehealth.demo`.
- [x] **Consent Verified:** Revisit `SH-P-0002` as `doctor1@securehealth.demo`—document list is now accessible.
- [x] **Consent Revocation:** As `patient2@securehealth.demo`, click "Revoke". Verify Doctor 1 is immediately blocked again.
- [x] **Consent History:** Open `/consent/<id>/history/` and verify chronological timeline shows both `Granted` and `Revoked` events.
- [x] **PDF Report Generation:** Click "Generate PDF Report" on an authorized patient—verify PDF downloads with disclaimer banner and patient details.
- [x] **Audit Log Review:** Log in as `admin@securehealth.demo`, open `/audit/`, and verify all login, grant, revoke, download, and denial events are logged.

---

## Security Architecture Summary

```
                      [ Client Browser ]
                              │
                    (HTTP GET/POST + CSRF)
                              ▼
                 [ _require_auth View Guard ]
                              │
               (Role-Based Access Control - RBAC)
                              ▼
            [ _can_access_patient() Assignment Check ]
                              │
               (Purpose-Specific Consent Engine)
                              ▼
             [ _has_consent() Patient Consent Check ]
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
[ Document Vault ]    [ ReportLab PDF ]     [ Consent Manager ]
 (media_private/)       (In-Memory Bytes)    (Append-Only Events)
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
                 [ AuditLog Security Logger ]
                              ▼
                   [( SQLite Database )]
```

---

## Project Structure

```text
patient_data_sec/
├── core/
│   ├── admin.py                      # Django admin for 8 models with immutable audit log
│   ├── apps.py                       # Application configuration
│   ├── forms.py                      # DocumentUploadForm, ShareDocumentForm, ConsentGrantForm
│   ├── management/
│   │   └── commands/
│   │       └── seed_demo.py          # Seeder for demo users, patients, docs, and consents
│   ├── migrations/
│   │   ├── 0001_initial.py           # UserProfile, Role, OTPToken, PatientProfile
│   │   ├── 0002_auditlog_...py       # MedicalDocument, SharingGrant, AuditLog
│   │   └── 0003_consentgrant_...py   # ConsentGrant, ConsentEvent
│   ├── models.py                     # All domain models and status enumerations
│   ├── templatetags/core_tags.py     # Template filter helpers
│   ├── tests.py                      # 97 automated unit and integration tests
│   ├── urls.py                       # 17 namespaced URL routes
│   └── views.py                      # Authentication, RBAC, documents, consent, and PDF views
├── docs/
│   └── TECHNICAL_MANUAL.md           # Comprehensive 16-section technical documentation manual
├── media_private/                    # Local storage for private medical documents
├── secure_health/
│   ├── settings.py                   # Project settings, WhiteNoise, media configuration
│   ├── urls.py                       # Root route configuration delegating to core
│   └── wsgi.py                       # WSGI application entrypoint
├── templates/
│   ├── base.html                     # Responsive navbar, role indicators, feedback alerts
│   └── core/
│       ├── audit_log.html            # Admin audit log dashboard with filters
│       ├── consent_history.html      # Chronological consent event timeline
│       ├── consent_manage.html       # Patient consent manager (grant form + active/revoked tables)
│       ├── dashboard.html            # Role-tailored user home view
│       ├── document_list.html        # Medical document listing
│       ├── document_upload.html      # Document upload with file validation
│       ├── home.html                 # Public home landing page
│       ├── login.html                # Step 1: Demo email entry & OTP panel
│       ├── patient_detail.html       # Complete patient chart view
│       ├── patient_list.html         # Directory view
│       ├── share_document.html       # Doctor document sharing form
│       ├── share_verify.html         # Recipient share OTP challenge
│       └── verify.html               # Step 2: Login OTP verification
├── db.sqlite3                        # SQLite database
├── manage.py                         # Django CLI wrapper with automatic venv detection
├── README.md                         # Project documentation and quick start
└── requirements.txt                  # Pinned dependencies
```
