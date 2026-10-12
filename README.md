# RPAS Documentation & Specifications Index

Welcome to the central documentation and specifications repository for the Real Property Assessment System (RPAS).

---

## Directory Structure

```text
docs/
├── architecture/                     # System-wide concepts, database schemas, numbering rules
│   └── rpas_numbering_relationships.md  # District, Barangay, SMV, TD No., and PIN specs & DB mappings
│
└── modules/                          # Feature and module-specific specifications
    └── reports/                      # Reporting module
        └── list_of_transactions/     # List of Transactions (Assessment & Cancellation) Report
            ├── report_spec.md        # General report specifications & business rules
            ├── api_spec.md           # Backend Laravel endpoint, request, and service spec
            └── frontend_spec.md      # React UI, filter dialog, document generation, and printing
```

---

## Guidelines for Adding Documentation

1. **Architecture & Cross-Cutting Docs** (`docs/architecture/`):
   - Place system-wide standards, data dictionaries, global numbering specifications, and core workflow diagrams here.
2. **Module Specifications** (`docs/modules/<module-name>/`):
   - Group module-level docs by feature/domain.
   - For complete feature specs, consider splitting into:
     - `spec.md` or `report_spec.md` (Overview and functional requirements)
     - `api_spec.md` (Backend request/response, validation, database queries)
     - `frontend_spec.md` (React components, state management, UI flows)
