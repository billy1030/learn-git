# MDS — Frontend UI Techniques & Design Patterns Guide
# (物料資料系統 — 前端 UI 設計與技術規範說明書)

> **Document Version:** v1.0.1  
> **Last Updated:** August 2026  
> **Target Audience:** Frontend Engineers, UI/UX Designers, and Technical Leads

---

## 🎨 UI Architecture Overview

MDS follows a modern, enterprise-grade UI engineering philosophy designed for **instant visual feedback, mobile usability on construction sites, precision physical printing, and defensive safety controls**.

```
┌───────────────────────────────────────────────────────────────────────────────────┐
│                             MDS UI ENGINEERING STACK                              │
├───────────────────────────────────────────────────────────────────────────────────┤
│  ⚡ Zero-Blank-Screen Boot    │ Native HTML/CSS Inline Pre-Boot Skeleton           │
│  📱 Mobile-First Responsive   │ Adaptive Card/Table Transforms & Collapsible Nav  │
│  🖨️ Precision Print CSS Engine│ @media print 2x4 Batch Sticker Formatting          │
│  🏎️ Stale-While-Revalidate    │ TanStack Query Memory Caching & Zero-Flicker Nav   │
│  🌐 Real-Time Dynamic i18n   │ Seamless Runtime Language Re-Rendering (No Reload) │
│  🛡️ Defensive Visual States   │ Frosted Glass Modals, Loading Locks, Status Badges │
│  📈 Visual Lifecycle Timeline │ Vertical SVG Milestone Tracking & Attachment Previews│
└───────────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. ⚡ Instant HTML Pre-Boot Skeleton (Zero Blank Screen)

* **Problem**: Single Page Applications (SPAs) typically show a blank white page for 1–2 seconds while downloading heavy JavaScript bundles.
* **Technique**:
  * Pure inline CSS and SVG branding embedded directly inside `<div id="root">` within `index.html`.
  * Renders at **millisecond zero (< 10ms)** before React even initializes.
* **Implementation Details**:
  ```html
  <div id="root">
    <div style="min-height: 100vh; display: flex; flex-direction: column; align-items: center; justify-content: center; background-color: #f8fafc;">
      <div style="width: 48px; height: 48px; border-radius: 12px; background: linear-gradient(135deg, #2563eb, #3b82f6); display: flex; align-items: center; justify-content: center; box-shadow: 0 4px 12px rgba(37,99,235,0.25);">
        <svg style="width: 26px; height: 26px; fill: white;" viewBox="0 0 24 24">...</svg>
      </div>
      <div style="width: 32px; height: 32px; border: 3px solid #e2e8f0; border-top-color: #2563eb; border-radius: 50%; animation: mds-spin 0.75s linear infinite;"></div>
    </div>
  </div>
  ```

---

## 2. 🖨️ Precision Print CSS Engine (`@media print`)

* **Feature**: **2x4 Batch Sticker Print (`/admin/materials/batch-print`)**.
* **Techniques**:
  * Dedicated `@media print` stylesheets with exact physical dimensions.
  * Multi-column CSS Grid layout (`grid-template-columns: repeat(2, 1fr)`).
  * Strict page break boundaries (`break-inside: avoid; page-break-inside: avoid`).
  * `@page { margin: 8mm; size: A4 portrait; }` ensures QR codes print at exact physical paper scale without truncation across standard laser sticker sheets.

---

## 3. 📱 Mobile-First Adaptive Responsive Layouts

* **Techniques**:
  * **Dual-Mode Data Display**: Wide desktops display dense, sortable data tables; mobile devices automatically transform rows into touch-friendly vertical cards.
  * **Collapsible Drawer & Hamburger Menu**: Smooth slide-down navigation menu optimized for one-handed operation on mobile phones.
  * **Dynamic Viewport Units**: Uses modern `dvh` / `min-h-screen` viewport units to prevent layout jumps caused by mobile Safari / Chrome collapsing address bars.

---

## 4. 🏎️ Optimistic UI & Stale-While-Revalidate (SWR)

* **Technique**: Powered by **TanStack React Query v5**.
* **Benefits**:
  * When navigating between administrative tabs (`/admin/materials` ➔ `/admin/audit` ➔ `/admin/settings`), previously fetched data displays **instantly (0ms)** from memory.
  * Silent background re-fetching verifies and patches any updated server state without UI flickering or layout shifts.
  * Configured with strategic `staleTime` (e.g. 5 minutes for authentication, 10 minutes for system settings).

---

## 5. 🛡️ Defensive UI & Feedback States

* **Micro-Interactions & Safety Controls**:
  * **Async Action Locks**: Form submit buttons automatically transition to spinning states and disable themselves to prevent accidental double-submission during network latency.
  * **Frosted Glass Modal Backdrops (`backdrop-blur-sm bg-slate-900/40`)**: Used in sensitive modal windows (Change Password, 2FA Setup, Delete Confirmations) to focus user attention and prevent background clicking.
  * **Visual State Badging**:
    * Frozen Material: High-contrast blue warning pill (`bg-blue-50 text-blue-700 border-blue-200`).
    * Sample Tested: Emerald success pill (`bg-emerald-50 text-emerald-700 border-emerald-200`).
    * Inactive/Deleted: Amber warning indicator.

---

## 6. 🌐 Real-Time Runtime Multi-Lingual Switching (Zero Page Reload)

* **Techniques**:
  * React `useTranslation` hook integrated directly into the component reactive tree.
  * Switching between **繁體中文**, **簡體中文**, and **English** triggers immediate virtual DOM re-renders without full page reload.
  * Date stamps, timestamps, and number formatting automatically format according to the active locale.

---

## 7. 🏷️ Dynamic Version & Build Hash Stamping

* **Techniques**:
  * Dynamic Rollup build-time constant injection via `vite.config.ts`.
  * **Top Header**: Clean version badge (`v1.0.1`).
  * **Bottom Footer**: Full diagnostics signature (`Version 1.0.1 (09a6929)`).
  * Gives operators instantaneous visual confirmation of whether the running code matches the latest Git commit.

---

## 8. 📈 Visual Material Lifecycle & Inspection Timeline

* **Feature**: **`TrackingTimeline.tsx`** component.
* **Techniques**:
  * Connected vertical SVG timeline with color-coded milestone nodes representing the entire material supply chain (Production ➔ Lab Testing ➔ Dispatch ➔ Acceptance).
  * Direct attachment preview drawers for PDF test certificates, photos, and inspection sign-offs.

---

## 📚 Cross-Reference Documentation
* Technology Stack Specifications: [tech-stack.md](tech-stack.md)
* System Architecture Whitepaper: [architecture-whitepaper.md](architecture-whitepaper.md)
* Database Schema & Relationships: [SCHEMA.md](SCHEMA.md)
* Main System Guide: [README.md](README.md)
