# Access forms and reports web migration

This document inventories representative Access forms and reports from `combinedForms.txt` and `combinedReports.txt` and outlines how to reproduce them in a modern web stack (React + React Router + React Hook Form + CSS grid/print styles). It focuses on key layouts, validation, and navigation behaviors that were previously encoded in Access properties (PopUp, AutoCenter, DefaultView, etc.).

## Framework choices

- **React** for componentized views that mirror Access forms and subforms.
- **React Router** for routing and modal overlays that replicate `PopUp`, `Modal`, and `AutoCenter` behaviors.
- **React Hook Form** (or Formik) for field registration, validation rules, and error messaging.
- **CSS grid/flex + `@media print`** for recreating datasheet-like layouts and print-friendly report rendering.

## Forms inventory and HTML mapping

### `frmCmpy_Companies` (company directory list)
- **Access behavior:** Popup list with hidden navigation and record selectors, vertical scroll bars, and a single-form view over `tblCmpy_Companies`, ordered by `companyID`; close logic restores the previously selected company when the active selection changes.【F:combinedForms.txt†L104150-L104979】
- **Key controls → HTML equivalents:**
  - `cmpyName` textbox doubles as an opener for the detail form; map to a clickable cell/button that routes to `/app/companies/:id` while persisting selection context (`SetSelected`/`SaveAllCurrentSettings`).【F:combinedForms.txt†L104742-L104779】【F:combinedForms.txt†L104967-L104970】
  - Partner and country dropdowns (`partnerID`, `countryID`) pull from lookup tables; recreate with async select components populated from partner and country endpoints, including combined name display formatting.【F:combinedForms.txt†L104705-L104737】【F:combinedForms.txt†L104911-L104939】
  - Status/metadata fields such as VAT status, tax number, payroll payday, manager/CEO, default cost center, and active flag should render as columns in a responsive table or grid with right alignment for numeric fields.【F:combinedForms.txt†L104661-L104909】
- **Navigation mapping:** Render as a searchable/paged table route (e.g., `/app/companies`) with row click/CTA opening the company detail modal/route. Use router guards or context to mirror the Access close handler that resets the active company when the form is dismissed.【F:combinedForms.txt†L104967-L104979】

### `frmCmpy_Company` (company detail with tabbed subforms)
- **Access behavior:** Popup single-form editor with hidden navigation/record selectors; uses a tab control to group registration data, accounting defaults, emails, cost centers, and payment means/bank accounts into subforms, all linked on `companyID` (defaulted from `currentCompanyID()`).【F:combinedForms.txt†L104992-L105009】【F:combinedForms.txt†L105451-L105776】【F:combinedForms.txt†L105781-L105793】
- **Tab/page mapping → React layout:**
  - **“Osnovni podatki”**: hosts subform `frmCmpyRegistrationData` → render a registration/profile section component bound to company master data.【F:combinedForms.txt†L105477-L105520】
  - **“Racunovodstvo”**: subform `frmCmpyDefaultValues` → accounting defaults component for fiscal settings, posted via the company API.【F:combinedForms.txt†L105525-L105569】
  - **“e-Mails”**: subform `frmCmpyEmails` with its own label → reusable email contacts table with add/edit rows tied to the company ID.【F:combinedForms.txt†L105573-L105639】
  - **“SM/oddelki”**: subform `frmCmpyCostCenters` plus label → cost center grid; ensure master-detail linkage via `companyID` in data fetching/mutations.【F:combinedForms.txt†L105640-L105704】
  - **“Pl. sr.” (payment means/banks)**: stacked subforms `frmCmpy_MeansOfPayments` and `frmAccPmtPaymentJournalNames` → payment instruments list and journal name selector; use split panes or stacked cards with shared company context.【F:combinedForms.txt†L105707-L105776】
- **Actions and routing:** Include a header-level “view original” action that triggers document retrieval (`OpenScan`)—map to a toolbar button calling a download/view endpoint. Present the form as a routed modal (`/app/companies/:id`) or dedicated page with tabs mirroring the Access `TabCtl` pages and preserve popup centering with modal styling.【F:combinedForms.txt†L105825-L105903】

### `frmApp_Login` (credentials popup)
- **Access behavior:** Configured as `PopUp` + `Modal` with disabled chrome (`ControlBox`, `MaxButton`, `MinButton`, `CloseButton`) and no scroll bars; centered via `AutoCenter`.【F:combinedForms.txt†L91968-L91999】
- **Web mapping:** Render as a React modal dialog (`<dialog>` or portal) that traps focus, hides background scroll, and uses CSS to center. Remove window controls and prevent backdrop dismissal to mirror the modal requirement.
- **Fields/controls:** Username/password inputs with submit; wire to validation that enforces non-empty password and error message parity with Access message boxes.
- **Navigation:** On success, redirect to the main shell route and close the modal; on cancellation, keep the router at `/login` until valid credentials are supplied.

### `frmApp_OfficeMainForm` (main shell)
- **Access behavior:** Single-form `DefaultView` with navigation controls hidden (`RecordSelectors`, `NavigationButtons`) and no dividing lines to present a clean dashboard container.【F:combinedForms.txt†L93524-L93546】
- **Web mapping:** Use a layout route (e.g., `/app`) that renders a persistent header/sidebar with nested routes for functional areas. Replace Access subform tabs with router sub-routes (e.g., `/app/purchase`, `/app/accounting`).
- **Routing flow:** The login modal transitions to this shell; primary navigation uses router links instead of Access buttons, while preserving role-based visibility via conditional rendering.

### `frmPurchaseInvoice` (single purchase invoice editor)
- **Access behavior:** `PopUp` with `AutoCenter`, no scroll bars, and single-record `DefaultView` intended as a focused editor.【F:combinedForms.txt†L279289-L279314】
- **Key controls → HTML equivalents:**
  - `purchaseInvoiceNumber` textbox with autogenerated default value → `<input>` bound to form state; lock or mark read-only once persisted; surface helper text showing the generated number.【F:combinedForms.txt†L280107-L280118】
  - `purchaseInvoiceAmount` textbox (numeric, standard format) → number input with currency/precision validation; align right and add thousands separators on blur.【F:combinedForms.txt†L280120-L280150】
  - `purchaseInvoiceDate` textbox with before/after update handlers → date picker input with required/valid date validation; trigger recalculations or period checks on change.【F:combinedForms.txt†L280165-L280208】
  - Additional metadata fields like `fiscalYear` defaulting to `CurrentFiscalYear()` → hidden/defaulted inputs populated from the active company context.【F:combinedForms.txt†L280211-L280227】
- **Layout:** Use CSS grid with labeled columns to mirror the label/textbox pairs shown in the Access definition; group header rectangle (`Box25`) can become a styled toolbar with actions (Save, Post, Print).
- **Validation/parity:** Port Access event logic into form validations (e.g., reject missing dates, enforce positive amount). Where Access used message boxes, surface inline errors or toast notifications.
- **Navigation:** Open the editor as a modal route (`/app/purchases/:id/edit`) so users can return to list views without a full page reload.

### `frmPurchaseInvoices` (list view)
- **Access hints:** Form-level configuration and filter code in the same file set filters like `purchaseInvoiceNumber Like '*…*'` and open `frmPurchaseInvoice` for edits.【F:combinedForms.txt†L287716-L290072】
- **Web mapping:** Implement as a paginated/searchable table route (`/app/purchases`) with column filters. Row click opens the modal editor route; reuse the search string to update the query params instead of Access’s `Me.Filter`.

## Reports inventory and print views

### `rptDocAcc_PI` (Book of received invoices)
- **Access behavior:** Print-oriented layout (`LayoutForPrint`), opened as a `PopUp` and centered, grouped by date with `GrpKeepTogether` to avoid page breaks in group headers.【F:combinedReports.txt†L66609-L66634】
- **Record source:** `qryDocAcc_PI`; caption “Knjiga prejetih racunov.”【F:combinedReports.txt†L66641-L66666】
- **Web mapping:** Create a React report view (`/reports/purchase-invoices`) that renders a read-only table grouped by invoice date. Use CSS print styles to fix widths (≈16,110 twips ≈ 11.2in) and enforce page-break-inside avoidance on group sections. Provide a “Print” button that invokes `window.print()` and sets `@page` margins to mimic Access defaults.

### `rptDoc_PurchaseOrder` (Purchase order printout)
- **Access behavior:** Print layout popup with grouping and date-based ordering; width ~10,773 twips to target a portrait page.【F:combinedReports.txt†L75002-L75025】
- **Record source:** company/registration data, cost centers, and purchase order lines referenced in the name map block.【F:combinedReports.txt†L75033-L75050】
- **Web mapping:** Build a printable React component that stacks company header, supplier block, and line items in a two-column grid. Use `@media print` rules for typography and ensure totals/footer sections use `position: sticky` or duplicated headers per page for readability.

## Navigation and modal parity

| Access property | Web equivalent | Notes |
| --- | --- | --- |
| `PopUp`, `Modal` | Modal routes with focus trapping and backdrop | Prevent background scroll; close via explicit actions only. |
| `AutoCenter` | CSS flex/grid centering for dialogs | Apply `align-items:center; justify-content:center; height:100vh` on modal containers. |
| `DefaultView = 0` (Single Form) | Single-record editor view | Map to detail routes or forms with isolated state. |
| Hidden `RecordSelectors`/`NavigationButtons` | Remove default list affordances | Provide toolbar buttons (Save, Cancel, Print) and router back links instead. |
| `LayoutForPrint`/`GrpKeepTogether` | Print media queries + `page-break-inside: avoid` | Ensure grouping rows and totals stay together on printed pages. |

## Implementation checklist

1. **Routing skeleton:** `/login` → `/app` shell → feature routes (`/app/purchases`, `/app/purchases/:id/edit`, `/reports/purchase-invoices`, `/reports/purchase-orders`).
2. **Shared layout:** Shell component hosts navigation and renders modal routes for popups; include a global context for `currentCompanyID` and `CurrentFiscalYear()` defaults.
3. **Forms:** Build controlled components for `frmPurchaseInvoice` fields with default generators and validation mirroring Access events; add list filtering for `frmPurchaseInvoices`.
4. **Reports:** Create read-only, print-styled pages for `rptDocAcc_PI` and `rptDoc_PurchaseOrder` with data fetched from equivalent queries/endpoints.
5. **Print readiness:** Add a print stylesheet that hides navigation, sets margins, and preserves grouping integrity.

This plan provides the mapping from Access artifacts to React components, preserving workflows (login → shell → modal editors) and printable outputs while embracing web-native routing and validation.
