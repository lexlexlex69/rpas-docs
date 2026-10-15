# RPAS Bulk General Revision Specification

## 1. Overview
The **Bulk General Revision** module provides local government assessors with the capability to perform mass/bulk general revision of real properties by Barangay and Schedule of Market Values (SMV). When transitioning from an older SMV (e.g. 2020) to a new SMV (e.g. 2027), unit values across all land, building, and machinery classifications must be systematically recomputed, new Tax Declarations generated, predecessor records cancelled by transaction, and relationship audit trails maintained.

---

## 2. Queue & Background Execution Architecture

### 2.1 Why Asynchronous Background Jobs are Required
Bulk assessment updates for dozens, hundreds, or thousands of properties carry substantial processing overhead:
- Fetching and parsing appraisal items, building floor/roofing specs, plants/trees.
- Matching unit values against the target SMV's `rpas_appraisal_categories`.
- Recomputing base market values, stripping allowances, depreciation percentages, and assessed values.
- Generating next sequential Tax Declaration numbers via `RpasTdNumberTrackerService`.
- Inserting `rpas_td_details`, `rpas_relationships`, `rpas_td_statuses`, and appraisal records in `DB::transaction`.

Executing this workload synchronously across an HTTP connection risks:
1. Gateway/Reverse Proxy timeouts (e.g. Cloudflare / Nginx 30s-60s timeout).
2. Client disconnection or network drops aborting the request mid-execution, leaving orphan or partially revised batches.
3. High browser memory consumption and frozen UI.

### 2.2 Queue Processing Model
1. **Initiation**: The frontend submits `POST /api/rpas/general-revision/bulk` containing `{ td_ids: [...], target_smv_id, rpas_barangay_id, source_smv_id }`.
2. **Validation**: The backend enforces that `target_smv_id` is the **immediate sequential successor** of `source_smv_id` (`target_smv.parent_id === source_smv_id`). Attempting to skip SMV versions (e.g., 2008 straight to 2027) or revising from the latest SMV is rejected with HTTP 422.
3. **Batch Record**: The API creates an entry in `rpas_general_revision_batches` with `status = 'pending'`, `total_count = count(td_ids)`, and dispatches `ProcessBulkGeneralRevisionJob` to Laravel's queue.
4. **HTTP 202 Accepted**: The API immediately returns `{ success: true, batch_uuid, batch_id, total_count }`.
5. **Worker Execution**: The queue worker handles `ProcessBulkGeneralRevisionJob`.
   - Each property is processed in an individual `DB::transaction(...)`.
   - Successful property: updates predecessor status to `CANCELLED`, links parent/child relationship in `rpas_relationships`, generates new `ACTIVE` TD with updated unit values. Increments `processed_count` and `success_count`.
   - Failed property: catches exception, increments `failed_count`, appends failure details `{ td_id, td_number, error_message }` into the batch `errors` JSON column.
6. **Real-time Progress Polling**: The frontend polls `GET /api/rpas/general-revision/batch/{id}` every 2 seconds to render a live progress modal with a percentage bar and count summary.

---

## 3. Database Schema

### Table: `rpas_general_revision_batches`
| Column | Type | Description |
|---|---|---|
| `id` | BIGINT UNSIGNED AUTO_INCREMENT | Primary Key |
| `batch_uuid` | VARCHAR(36) UNIQUE | Unique UUID for frontend polling |
| `user_id` | BIGINT UNSIGNED | Assessor who triggered the batch |
| `source_smv_id` | BIGINT UNSIGNED | Source SMV ID (e.g., 2020) |
| `target_smv_id` | BIGINT UNSIGNED | Target SMV ID (e.g., 2027) |
| `rpas_barangay_id` | BIGINT UNSIGNED | Selected Barangay ID |
| `total_count` | INT UNSIGNED | Total number of properties queued |
| `processed_count` | INT UNSIGNED DEFAULT 0 | Number of properties processed so far |
| `success_count` | INT UNSIGNED DEFAULT 0 | Successful creations |
| `failed_count` | INT UNSIGNED DEFAULT 0 | Failed creations |
| `status` | ENUM('pending', 'processing', 'completed', 'failed') | Current batch status |
| `errors` | JSON NULLABLE | Array of failure objects `{ td_id, td_number, error }` |
| `created_at` / `updated_at` | TIMESTAMP | Audit timestamps |

---

## 4. Frontend Specifications

### 4.1 Route & Navigation
- Route: `/rpas/general-revision`
- Navigation Label: `"General Revision"`
- Position: RPAS Top-Level navigation alongside `Real Property File`, `Dashboard`, and `Notice of Cancellation`.

### 4.2 Top Section Layout (`GeneralRevisionTopSection.jsx`)
- **Retained**:
  - Barangay Autocomplete Filter
  - Source SMV Autocomplete Filter
  - Search Input Box
  - Reset Filters Button
- **Hidden / Removed**:
  - ❌ `Discovery` Button
  - ❌ `DataGridFilterMenu` Button
  - ❌ `RPFNotifications`
  - ❌ More Options (`<MoreVertIcon />`) Button
- **Added**:
  - ✅ **Selected Count Indicator**: Displays current selection count (e.g. `Selected: 15 / Total: 120`).
  - ✅ **"Proceed Gen. Rev." Button**:
    - Disabled when `selectedCount === 0`.
    - Clicking opens `ProceedGeneralRevisionDialog`.

### 4.3 Data Table (`InfiniteDataGrid`)
- **Single-Row Individual TD Mode (`individual_td: true`)**:
  - The General Revision module passes `individual_td: true` in the `POST /api/rpas/real-property-files/get-real-property` payload.
  - When `individual_td === true`, the API does **not** split/group Tax Declarations by actual use. Each Tax Declaration is returned as a single, distinct row with aggregated TD totals (avoiding duplicate IDs, split rows, or blanked-out cells).
- `checkboxSelection={true}`: Native selection column.
- Row click or checkbox click marks row as selected.
- No bottom summary section (`RPFBottomSection`) on row click.
- Column **"Gen. Rev. Status"**:
  - `is_latest_smv === true`: Blue chip **"Latest SMV"** with tooltip: *Property is on the latest SMV revision. No General Revision is applicable.*
  - `has_general_revision === true`: Green chip **"Revised"** with tooltip showing revised TD number in the immediate next SMV series.
  - `has_general_revision === false` (and not latest): Orange/Neutral chip **"Pending"** (eligible for General Revision to next SMV).
- `isRowSelectable`: Disabled for rows where `is_latest_smv === true` or `has_general_revision === true` to prevent invalid or duplicate revisions.

### 4.4 Modals
1. **`ProceedGeneralRevisionDialog`**:
   - Confirms total selected TD count.
   - **Target SMV Selection**: Strictly locked to the **immediate successor SMV** (`target_smv.parent_id === source_smv.id`). Version jumping (e.g. 2008 straight to 2027) is prohibited.
   - If current SMV is the latest SMV (`is_latest === true`), dialog warns that no revision is available and disables action.
   - "Start General Revision" trigger dispatches background queue job.
2. **`GeneralRevisionProgressDialog`**:
   - Live progress bar (`X of Y properties processed`).
   - Success / Failure counters.
   - Error log accordion if any properties encountered issues.
   - Refreshes table upon completion.
