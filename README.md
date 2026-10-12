# RPAS Documentation & Specifications Index

Welcome to the central documentation and specifications repository for the Real Property Assessment System (RPAS).

---

## Directory Structure

```text
docs/
├── architecture/                     # System-wide concepts, database schemas, numbering rules
│   ├── rpas_numbering_relationships.md  # District, Barangay, SMV, TD No., and PIN specs & DB mappings
│   ├── rpas_transaction_processes.md    # Transactions list, TD lifecycle, segregation, consolidation & NOC
│   └── rpas_td_statuses.md              # Two-tier TD status architecture (active/cancelled vs validated/approved)
│
└── modules/                          # Feature and module-specific specifications
    └── reports/                      # Reporting module
        ├── list_of_transactions/     # List of Transactions (Assessment & Cancellation) Report
        │   ├── report_spec.md        # General report specifications & business rules
        │   ├── api_spec.md           # Backend Laravel endpoint, request, and service spec
        │   └── frontend_spec.md      # React UI, filter dialog, document generation, and printing
        ├── tax_map_control_roll/     # Tax Map Control Roll (TMCR) Report
        │   ├── api_spec.md           # Backend API hierarchy, active-only filter, and response schema
        │   └── frontend_spec.md      # React document layout, visual header boxes, and LOT PIN table
        └── record_of_assessment_and_cancellation/ # Record of Assessment & Cancellation (ROA & RCA)
            └── spec.md               # Summary for Each Class report spec, table layout & sidebar integration
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
