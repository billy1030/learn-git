# Reusable Code & UI Components Catalog

A granular, production-ready reference of reusable architectural patterns, backend micro-utilities, and atomic frontend widgets extracted from the **MDS (Material Data System)** codebase.

---

## 1. Master Component Breakdown (5-Level Atomic Hierarchy)

```
Level 1: Micro-Atoms & Badges (Single-purpose visual tokens & state indicators)
Level 2: Interactive Cell & Form Molecules (Single-field inline inputs & auto-mutators)
Level 3: Feedback & Dialog Molecules (Notifications, confirmations & prompt screens)
Level 4: Print & Media Engines (Physical layout wrappers & SVG/Canvas rasterizers)
Level 5: Domain Organisms & Panels (Multi-field timeline streams & asset managers)
Level 6: Backend Micro-Utilities & Security Engines (Cryptographic & data integrity helpers)
```

---

## 2. Master Component & Code Gadget Matrix

| Level | Component / Utility | Category | File Location | Key Tech & Standards | Reusability | Key Capabilities |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- |
| **L1** | **`ResultBadge`** | Quality Token | [`ResultBadge.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/ResultBadge.tsx) | React, Tailwind, Lucide | ⭐️⭐️⭐️⭐️ (High) | Tri-state pill: **Pass** (Emerald), **Fail** (Red), and **Pending** (Slate). |
| **L1** | **`FrozenBadge`** | Security Alert | [`FrozenBadge.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/FrozenBadge.tsx) | Tailwind, Lucide `ShieldAlert` | ⭐️⭐️⭐️⭐️ (High) | Prominent administrative hold banner displaying quarantine reason. |
| **L1** | **`SerialRow`** | Barcode Display | [`SerialRow.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/SerialRow.tsx) | Font Mono, CSS `select-all` | ⭐️⭐️⭐️⭐️ (High) | Single-click selection wrapper for barcode scanner / clipboard inputs. |
| **L1** | **`BrandBar`** | Scan Header | [`BrandBar.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/BrandBar.tsx) | Minimalist Navbar, i18n | ⭐️⭐️⭐️⭐️ (High) | Lightweight top-bar for public scan pages with language switch widget. |
| **L2** | **`UserDisplayNameCell`** | Inline Table Editor | [`UserDisplayNameCell.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/UserDisplayNameCell.tsx) | React Query Mutation, Lucide | ⭐️⭐️⭐️⭐️⭐️ (High) | Hover-reveal edit icon; `Enter` to save, `Esc` to cancel; auto-focus. |
| **L2** | **`UserRoleSelect`** | Instant Mutation Dropdown | [`UserRoleSelect.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/UserRoleSelect.tsx) | React Query Mutation | ⭐️⭐️⭐️⭐️ (High) | Self-mutating `<select>` that auto-disables during network flight. |
| **L2** | **`PageSizeSelect`** | Pagination Select | [`AuditLog.tsx:L492`](file:///Users/billylam/ai/mds/frontend/src/pages/admin/AuditLog.tsx#L492) | React State, Tailwind | ⭐️⭐️⭐️⭐️⭐️ (High) | Dropdown for items per page (`10/20/50/100`); resets active page to 1. |
| **L2** | **`LanguageSwitcher`** | Global Toolbar | [`LanguageSwitcher.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/LanguageSwitcher.tsx) | `react-i18next`, Lucide | ⭐️⭐️⭐️⭐️ (High) | Accessible dropdown with `<span className="sr-only">` screen-reader label. |
| **L3** | **`ConfirmDialog`** | Modal Primitive | [`ConfirmDialog.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/ConfirmDialog.tsx) | WAI-ARIA `dialog`, Tailwind | ⭐️⭐️⭐️⭐️ (High) | Focus trapping, `Escape` key capture, backdrop dismissal, spin indicator. |
| **L3** | **`MaterialMaintenanceDeleteModal`** | Privileged Security Modal | [`MaterialMaintenanceDeleteModal.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/MaterialMaintenanceDeleteModal.tsx) | WAI-ARIA `dialog`, Lucide `AlertTriangle` | ⭐️⭐️⭐️⭐️ (High) | High-privilege modal with exact serial match validation and mandatory audit reason prompt. |
| **L3** | **`Toast`** | Status Toast | [`Toast.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/ui/Toast.tsx) | CSS Animations, `aria-live` | ⭐️⭐️⭐️⭐️ (High) | Auto-dismissing fixed pill supporting `success`, `warning`, `info`. |
| **L3** | **`FrozenNotice`** | Quarantine Lock Screen | [`FrozenNotice.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/FrozenNotice.tsx) | React Router, `role="status"` | ⭐️⭐️⭐️⭐️ (High) | Full-screen alert screen for administrative holds or quarantined records. |
| **L3** | **`LoginPrompt`** | Public Scan CTA | [`LoginPrompt.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/LoginPrompt.tsx) | React Router | ⭐️⭐️⭐️⭐️ (High) | Callout linking to `/login?return_to=/m/:serial` with query encoding. |
| **L3** | **`EditControlsBanner`** | Contextual Action Bar | [`EditControlsBanner.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/EditControlsBanner.tsx) | React Router, Lucide | ⭐️⭐️⭐️⭐️ (High) | Floating admin toolbar rendered when authenticated users inspect public items. |
| **L4** | **`PrintLabelCard`** | Thermal Label Card | [`PrintLabelCard.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/PrintLabelCard.tsx) | CSS Print Media, `mm` units | ⭐️⭐️⭐️⭐️ (High) | Standard `70mm x 35mm` layout, `break-inside-avoid`, and QR fallback. |
| **L4** | **`PrintPreviewModal`** | A4 Batch Print Engine | [`PrintPreviewModal.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/PrintPreviewModal.tsx) | WAI-ARIA `dialog`, CSS Print | ⭐️⭐️⭐️⭐️ (High) | 8-label A4 layout preview with batch error tally and `window.print()`. |
| **L4** | **`QrPanel`** | SVG to PNG Rasterizer | [`QrPanel.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/QrPanel.tsx) | HTML5 Canvas, `XMLSerializer` | ⭐️⭐️⭐️⭐️⭐️ (High) | Rasterizes vector SVG to 512x512 PNG data URLs directly in the browser. |
| **L5** | **`TrackingTimeline`** | Connected Event Stream | [`TrackingTimeline.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/TrackingTimeline.tsx) | CSS Pseudo-elements, RBAC | ⭐️⭐️⭐️⭐️⭐️ (High) | Continuous vertical line, RBAC field masking, and stage-for-deletion UI. |
| **L5** | **`InspectionCard`** | Quality Summary | [`InspectionCard.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/InspectionCard.tsx) | `ResultBadge`, i18n date | ⭐️⭐️⭐️⭐️ (High) | Detailed inspection block with laboratory metadata and defect breakdown. |
| **L5** | **`FileAttachmentPanel`** | Asset Manager & Lightbox | [`FileAttachmentPanel.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/FileAttachmentPanel.tsx) | React Query, Lightbox Modal | ⭐️⭐️⭐️⭐️ (High) | Drag-and-drop queue with 10MB size checks and PDF/image preview modal. |
| **L6** | **`generatePreAuthToken`** | 2FA Handshake | [`two-factor.service.ts`](file:///Users/billylam/ai/mds/backend/src/services/two-factor.service.ts#L14) | HMAC-SHA256, Timing Safe | ⭐️⭐️⭐️⭐️⭐️ (High) | Signed 5-minute pre-auth token binding password verify to TOTP challenge. |
| **L6** | **`diffKeys`** | JSON Object Diff | [`audit.ts`](file:///Users/billylam/ai/mds/backend/src/lib/audit.ts#L26) | Pure TypeScript Function | ⭐️⭐️⭐️⭐️⭐️ (High) | Deep comparison returning sorted array of modified field keys. |
| **L6** | **`generateETag`** | OCC Concurrency | [`etag.ts`](file:///Users/billylam/ai/mds/backend/src/lib/etag.ts#L3) | SHA-256 Weak ETag | ⭐️⭐️⭐️⭐️⭐️ (High) | Fast entity version hashing: `W/"${hash}"`. |
| **L6** | **`escapeLikePattern`** | SQL Search Sanitizer | [`materials.service.ts`](file:///Users/billylam/ai/mds/backend/src/services/materials.service.ts#L107) | RegEx `/[%_\\]/g` | ⭐️⭐️⭐️⭐️⭐️ (High) | Neutralizes SQL wildcard injection in dynamic `LIKE`/`ILIKE` search queries. |
| **L6** | **`encodeRFC5987ValueChars`** | Unicode Filename Sanitizer | [`files.ts`](file:///Users/billylam/ai/mds/backend/src/lib/files.ts#L69) | RFC 5987 URI encoding | ⭐️⭐️⭐️⭐️ (High) | Formats `filename*="UTF-8''..."` headers for CJK file downloads. |
| **L6** | **`generateRecoveryCodes`** | 2FA Backup Tokens | [`recovery-codes.ts`](file:///Users/billylam/ai/mds/backend/src/lib/recovery-codes.ts#L7) | `crypto.randomBytes`, Argon2id | ⭐️⭐️⭐️⭐️ (High) | Generates `XXXX-XXXX-XXXX-XXXX` tokens with symbol-stripping verification. |
| **L6** | **`formatDate`** | Zero-Dep UTC Formatter | [`dates.ts`](file:///Users/billylam/ai/mds/backend/src/lib/dates.ts#L10) | `Intl.DateTimeFormat` (en-CA) | ⭐️⭐️⭐️⭐️ (High) | Timezone-safe `YYYY-MM-DD` formatting without heavy external libraries. |

---

## 3. Code Snippets & Implementation Guide

### 3.1. Inline Table Field Editor (`UserDisplayNameCell.tsx`)
```tsx
import { useState } from "react";
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { Edit2, Check, X } from "lucide-react";

export default function UserDisplayNameCell({ user }: { user: { id: number; username: string; display_name?: string } }) {
  const queryClient = useQueryClient();
  const [isEditing, setIsEditing] = useState(false);
  const [displayName, setDisplayName] = useState(user.display_name || user.username);

  const mutation = useMutation({
    mutationFn: (newName: string) => updateUser(user.id, { display_name: newName.trim() }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["users"] });
      setIsEditing(false);
    },
  });

  if (isEditing) {
    return (
      <div className="flex items-center gap-1.5">
        <input
          type="text"
          value={displayName}
          onChange={(e) => setDisplayName(e.target.value)}
          onKeyDown={(e) => {
            if (e.key === "Enter" && displayName.trim()) mutation.mutate(displayName);
            if (e.key === "Escape") setIsEditing(false);
          }}
          autoFocus
          className="px-2 py-1 text-xs border border-blue-400 rounded bg-white text-slate-900 focus:outline-none focus:ring-1 focus:ring-blue-500 w-36"
        />
        <button onClick={() => mutation.mutate(displayName)} disabled={mutation.isPending} className="p-1 rounded bg-emerald-50 text-emerald-700">
          <Check className="w-3.5 h-3.5" />
        </button>
        <button onClick={() => setIsEditing(false)} className="p-1 rounded bg-slate-100 text-slate-600">
          <X className="w-3.5 h-3.5" />
        </button>
      </div>
    );
  }

  return (
    <div className="group flex items-center gap-2">
      <span className="font-medium text-slate-900">{user.display_name || user.username}</span>
      <button onClick={() => setIsEditing(true)} className="opacity-0 group-hover:opacity-100 p-1 text-slate-400 hover:text-blue-600 transition">
        <Edit2 className="w-3.5 h-3.5" />
      </button>
    </div>
  );
}
```

---

### 3.2. Page Size Selector Widget (`PageSizeSelect.tsx`)
```tsx
interface PageSizeSelectProps {
  pageSize: number;
  onPageSizeChange: (size: number) => void;
  options?: number[];
}

export function PageSizeSelect({ pageSize, onPageSizeChange, options = [10, 20, 50, 100] }: PageSizeSelectProps) {
  return (
    <div className="flex items-center gap-1.5">
      <span className="text-slate-500 font-medium text-xs">Per page:</span>
      <select
        value={pageSize}
        onChange={(e) => onPageSizeChange(Number(e.target.value))}
        className="px-2.5 py-1 text-xs font-semibold bg-white border border-slate-300 rounded-lg shadow-2xs focus:ring-2 focus:ring-blue-500 focus:outline-none cursor-pointer"
        aria-label="Records per page"
      >
        {options.map((opt) => (
          <option key={opt} value={opt}>{opt}</option>
        ))}
      </select>
    </div>
  );
}
```

---

### 3.3. Client-Side SVG to PNG Rasterizer (`downloadSvgAsPng`)
```typescript
export function downloadSvgAsPng(svgElement: SVGSVGElement | null, filename: string, size = 512) {
  if (!svgElement) return;
  const svgData = new XMLSerializer().serializeToString(svgElement);
  const canvas = document.createElement("canvas");
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext("2d");
  const img = new Image();
  img.onload = () => {
    if (ctx) {
      ctx.fillStyle = "white";
      ctx.fillRect(0, 0, size, size);
      ctx.drawImage(img, 0, 0, size, size);
      const pngUrl = canvas.toDataURL("image/png");
      const downloadLink = document.createElement("a");
      downloadLink.href = pngUrl;
      downloadLink.download = filename;
      document.body.appendChild(downloadLink);
      downloadLink.click();
      document.body.removeChild(downloadLink);
    }
  };
  img.src = "data:image/svg+xml;base64," + btoa(unescape(encodeURIComponent(svgData)));
}
```

---

### 3.4. SQL Search Wildcard Sanitizer (`escapeLikePattern`)
```typescript
/**
 * Escapes %, _ and backslashes to prevent wildcard injection in SQL ILIKE/LIKE queries
 */
export function escapeLikePattern(str: string): string {
  return str.replace(/[%_\\]/g, "\\$&");
}
```

---

### 3.5. RFC 5987 Unicode Filename Encoder
```typescript
/**
 * Formats Content-Disposition headers for Unicode & CJK filenames with ASCII fallbacks
 */
export function encodeRFC5987ValueChars(str: string): string {
  return encodeURIComponent(str)
    .replace(/['()]/g, escape)
    .replace(/\*/g, "%2A")
    .replace(/%(?:7C|60|5E)/g, unescape);
}

export function sanitizeAsciiFilename(filename: string): string {
  return filename.replace(/[\r\n\x00-\x1f\x7f"\\]/g, "_").replace(/[^\x20-\x7e]/g, "_");
}
```
