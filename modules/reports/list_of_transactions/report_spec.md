# Feature Specification & AI Reference Prompt
## List of Transactions (Assessment and Cancellation) Report

---

## 1. Overview & Location
- **Location in UI**: [`casd-web/src/pages/private/rpas/reports/index.jsx`](file:///c:/Users/Administrator/Desktop/RPAS/Development/casd-web/src/pages/private/rpas/reports/index.jsx)
- **New Section**: **"Others"** (placed below the main report card generation grid)
- **New Report Card**: **"LIST OF TRANSACTIONS (ASSESSMENT AND CANCELLATION)"**
- **Action**: Clicking the card triggers a parameter filter dialog prior to generating the preview document.

---

## 2. Filter Modal Parameters

| Filter Field | Type / Control | Description |
| :--- | :--- | :--- |
| **SMV Year** | Dropdown / Select | General revision year (`rpas_smv_id`) |
| **Barangay** | Dropdown / Select | Barangay selection (or **All Barangays**) |
| **Month & Year** | Select / DatePicker | Filters TDs based on `rpas_td_sub_statuses.updated_at` |
| **Taxability** | Segmented / Radio | `ALL`, `TAXABLE`, `EXEMPTED` |
| **Is Validated** | Checkbox | Filter sub-status code `is_validated` (`rpas_sub_status_id = 1`) |
| **Is Approved** | Checkbox | Filter sub-status code `is_approved` (`rpas_sub_status_id = 2`) |

> **Backend Filter Logic Note**:
> When filtering by Month and Year, check `rpas_td_sub_statuses`:
> - `rpas_sub_status_id = 1` for **Validated** (using `updated_at` month & year)
> - `rpas_sub_status_id = 2` for **Approved** (using `updated_at` month & year)

---

## 3. Data Grouping & Business Rules

### Rule A: Classification vs. Actual Use Alignment
- **Market Value (MV)** & **Area**: Grouped under the property's **Classification** (Class).
- **Assessed Value (AV)**: Grouped under the property's **Actual Use**.

### Rule B: Cross-Classification Visual Tracer (e.g., TD `20-05867`)
- **Problem**: A TD has Class = `COMMERCIAL`, but Actual Use = `RESIDENTIAL`.
  - In **Commercial Land ROA**: Displays TD `20-05867`, No. RPU (`1`), Area, and Market Value. Assessed Value remains blank.
  - In **Residential Land ROA**: Displays the Assessed Value (`5,890`).
- **Solution / Feature Requirement**:
  - In the **Residential Land ROA** section, display the TD Number `20-05867` as well, formatted in a **light/muted color** (e.g., `#307eec` / italic tracer badge).
  - **No No. RPU for Tracers**: The tracer row must display a dash (`"—"`) under the **No. RPU** column (no RPU count) and must NOT be added to total RPU counts, as the physical RPU was already counted under Commercial.
  - This allows assessors to immediately trace where that Assessed Value originated from in Commercial without duplicating RPU counts.

### Rule C: Building Market Value Tiers
Group Buildings into two distinct rows/tiers based on Market Value:
1. **Over 175,000** (`Market Value > 175,000`)
2. **Less than 175,000** (`Market Value <= 175,000`)

### Rule D: RCA (Roll of Cancelled Assessment)
- Leave RCA columns **blank / dash** for the initial implementation.

---

## 4. Backend Category Constants Created

Created [`RpasCategoryConstant.php`](file:///c:/Users/Administrator/Desktop/RPAS/Development/casd-api/app/Constants/RpasCategoryConstant.php) under `app/Constants/`:

```php
// Backend Constant File: app/Constants/RpasCategoryConstant.php
namespace App\Constants;

class RpasCategoryConstant
{
    public const RESIDENTIAL  = 'RESIDENTIAL';
    public const COMMERCIAL   = 'COMMERCIAL';
    public const INDUSTRIAL   = 'INDUSTRIAL';
    public const AGRICULTURAL = 'AGRICULTURAL';
    public const TIMBERLAND   = 'TIMBERLAND';
    public const SPECIAL      = 'SPECIAL';

    public const RESIDENTIAL_CLASS_IDS = [1, 6, 12, 13];
    public const RESIDENTIAL_ACTUAL_USE_IDS = [1, 6, 11];

    public static function isResidentialClass(?int $classId): bool;
    public static function isResidentialActualUse(?int $actualUseId): bool;
    public static function getCategoryByClassId(?int $classId): string;
    public static function getCategoryByActualUseId(?int $actualUseId): string;
}
```

---

## 5. Complete AI Reference Prompt (Copy & Save for Future Use)

Below is the prompt you can save and copy whenever you request AI to build or update this report:

````markdown
You are an expert RPAS (Real Property Assessment System) developer.

Task: Implement / Update the "LIST OF TRANSACTIONS (ASSESSMENT AND CANCELLATION)" report feature.

1. UI & Filter Dialog Requirements:
   - File: `casd-web/src/pages/private/rpas/reports/index.jsx`
   - Location: Under a new section titled "Others", add a report card for "LIST OF TRANSACTIONS (ASSESSMENT AND CANCELLATION)".
   - Dialog Filters: SMV Year, Barangay, Month, Year, Taxability (All, Taxable, Exempted), and Checkboxes for Validated & Approved.
   - Status Filter Logic: Filter TDs by `rpas_td_sub_statuses.updated_at` (matching selected Month & Year), where `rpas_sub_status_id = 1` (Validated) and `rpas_sub_status_id = 2` (Approved).

2. Property & Table Breakdown Rules:
   - Land, Building, and Machinery are displayed in side-by-side columns, grouped under Major Categories (RESIDENTIAL, COMMERCIAL, INDUSTRIAL, AGRICULTURAL).
   - Market Value & Area are grouped by Classification (Class).
   - Assessed Value is grouped by Actual Use.

3. Cross-Classification Visual Tracer:
   - If a TD has Class = COMMERCIAL and Actual Use = RESIDENTIAL (e.g. TD 20-05867):
     - In COMMERCIAL ROA: Display TD number, No. RPU (`1`), Area, and Market Value. Assessed Value is empty.
     - In RESIDENTIAL ROA: Display the Assessed Value AND display the TD number in a light/muted color style (tracer badge) so users can trace its matching TD from Commercial.
     - **No No. RPU**: Tracer entries in Residential ROA must have **no No. RPU** (`"—"` / blank) and are excluded from total RPU counts to prevent duplicate counting.

4. Building Tier Grouping:
   - Group Buildings under "OVER 175,000" (> 175,000 Market Value) and "LESS THAN 175,000" (<= 175,000 Market Value).

5. Cancelled Assessment (RCA):
   - Leave RCA columns blank / unpopulated.

6. Backend Constants Reference:
   - Use `App\Constants\RpasCategoryConstant` for mapping Class IDs and Actual Use IDs to unified Major Categories (RESIDENTIAL, COMMERCIAL, INDUSTRIAL, AGRICULTURAL).
````
