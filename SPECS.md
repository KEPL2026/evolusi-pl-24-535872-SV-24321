# Project Requirements Document (PRD): Client-Side Invoice Generator App

**Document Version:** 1.0.0  
**Status:** Ready for Development  
**Target Stack:** Pure Client-Side (HTML5, CSS3, Alpine.js, htmx)

---

## 1. Executive Summary & Objective

The goal of this project is to build a lightweight, zero-backend, client-side web application for generating, previewing, and printing professional invoices. The application runs entirely within the browser, requiring no server-side rendering or database storage.

Users can dynamically input sender details, client information, invoice metadata, tax/discount configurations, and line items. The app automatically calculates itemized line totals, sub-totals, taxes, discounts, and grand totals in real-time, instantly rendering a pixel-perfect invoice preview that can be printed or saved as a PDF using standard browser print capabilities.

---

## 2. Architecture & Technical Constraints

* **Execution Context:** 100% Client-Side Single Page Application (SPA).
* **Dependencies (via CDN):**
  * `Alpine.js` (v3.x) — Primary reactive state management, line-item arrays, and arithmetic calculations.
  * `htmx` (v1.9.x) — UI event orchestration, seamless DOM manipulation, and action triggering.
* **Storage:** Browser `localStorage` for automatic state persistence across session reloads.
* **Export Mechanism:** Native browser `window.print()` combined with tailored `@media print` CSS rules.
* **Privacy & Security:** Zero data egress. All inputted customer data remains local to the user's browser session.

---

## 3. UI Layout Architecture

The application adopts a two-column responsive split view on desktop devices, which collapses into a stacked layout on smaller screens.

```
+---------------------------------------------------------------------------------------+
|  TOP BAR: Application Header | Action Controls [Reset Form] [Save Draft] [Print/PDF]  |
+---------------------------------------------------+-----------------------------------+
|  INPUT FORM PANEL (.no-print)                     |  LIVE PREVIEW PANE (#invoice-app) |
|                                                   |                                   |
|  [Section] Business & Client Details              |  +-----------------------------+  |
|  - Company Name, Address, Logo URL                |  | [COMPANY LOGO]  INVOICE #   |  |
|  - Client Name, Billing Address                   |  |                             |  |
|                                                   |  | Billed To:      Date:       |  |
|  [Section] Invoice Metadata                       |  | Client Details  Due Date:   |  |
|  - Invoice #, Issue Date, Due Date, Currency      |  |                             |  |
|                                                   |  | --------------------------- |  |
|  [Section] Line Items Manager                     |  | Item    Qty    Price  Total |  |
|  - Description, Qty, Unit Price                   |  | Item 1    2    $50.00 $100.00|  |
|  - [+ Add Row]  [Delete Row]                      |  |                             |  |
|                                                   |  | Subtotal:           $100.00 |  |
|  [Section] Adjustments                            |  | Tax (10%):           $10.00 |  |
|  - Tax Rate (%)                                   |  | Discount:            -$5.00 |  |
|  - Discount Rate (%) / Fixed Amount               |  | **Grand Total:       $105.00**|  |
|  - Notes / Payment Instructions                   |  +-----------------------------+  |
+---------------------------------------------------+-----------------------------------+
```

---

## 4. Functional Requirements

### 4.1. Header & Business Metadata Input
* **Sender Details:**
  * Company / Sender Name
  * Address Line 1 & Line 2
  * Contact Email & Phone Number
  * Business Logo URL (or base64 image upload)
* **Client Details:**
  * Client Company / Contact Person
  * Billing Address
  * Client Email
* **Invoice Metadata:**
  * Invoice Number (e.g., `INV-2026-001`)
  * Issue Date (default: Current Date)
  * Payment Due Date
  * Currency Selector (`$`, `€`, `£`, `¥`, etc.)

### 4.2. Dynamic Line Items & Real-Time Calculation
* **Line Item Table Attributes:**
  * Item Description (Text String)
  * Quantity (Numeric, non-negative, supports decimals)
  * Unit Price (Numeric, non-negative)
* **Dynamic Row Controls:**
  * Ability to append new line item rows dynamically.
  * Ability to remove specific rows with instant array updating.
* **Automated Arithmetic Formulas:**
  * $$	ext{Row Total} = 	ext{Quantity} 	imes 	ext{Unit Price}$$
  * $$	ext{Subtotal} = \sum 	ext{Row Totals}$$
  * $$	ext{Tax Amount} = 	ext{Subtotal} 	imes \left(rac{	ext{Tax Rate \%}}{100}
ight)$$
  * $$	ext{Discount Amount} = 	ext{Subtotal} 	imes \left(rac{	ext{Discount Rate \%}}{100}
ight)$$
  * $$	ext{Grand Total} = (	ext{Subtotal} - 	ext{Discount Amount}) + 	ext{Tax Amount}$$

### 4.3. Print & PDF Export Workflow
* **Trigger:** Primary action button calls `window.print()`.
* **Print Stylesheet (`@media print`):**
  * Automatically hides input panel, navigation header, action buttons, and instructions.
  * Adjusts preview area width to 100% of standard A4 / US Letter boundaries.
  * Removes background shadows, borders, and UI padding to ensure crisp rendering.
  * Enforces `page-break-inside: avoid` on total blocks and table rows to prevent awkward multi-page breaks.

### 4.4. Persistence & Reset
* Automatically serializes form state to `localStorage` on every keystroke/change.
* Loads existing stored data on initial page load.
* Provides a **"Reset Form"** modal/button to purge storage and re-initialize default values.

---

## 5. Non-Functional & Design Requirements

* **Performance:** Real-time updates must occur in under 16ms (60 FPS feel) with zero perceptible lag.
* **Cross-Browser Compatibility:** Full support on Chrome, Firefox, Safari, and Edge.
* **Accessibility:** Standard HTML form labels, ARIA tags on button icons, full keyboard navigation for line-item creation.
* **Typography & Styling:** Modern sans-serif typography (e.g., Inter, system-ui) for input forms, and clean print-optimized layout fonts.

---

## 6. Verification & Test Plan

1. **Calculation Accuracy Test:** Verify that fractions, multi-item quantities, tax percentages, and discounts sum accurately to two decimal places without floating-point precision errors (e.g., $0.1 + $0.2 rounding bugs).
2. **Dynamic UI Test:** Add 10+ line items, remove items from the middle of the array, and verify indices and totals adjust immediately.
3. **Print Layout Verification:** Execute print preview in Chrome and Firefox to ensure input controls are completely hidden and invoice content fits cleanly on an A4 page.
4. **State Persistence Test:** Enter custom values, refresh the browser window, and confirm all fields reload intact from `localStorage`.