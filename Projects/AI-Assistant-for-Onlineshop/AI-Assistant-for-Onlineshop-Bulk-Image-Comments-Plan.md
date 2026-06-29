---
title: "Bulk Add + Image + Comments Plan"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/gap-improvement-plan/bulk-image-comments-plan.md"
---

# Bulk Add + Image + Comments Plan

## Goal

Дэлгүүрийн бараа нэмэхийг хурдан болгох, Messenger дээр зурагтай харилцааг дэмжих, мөн Facebook comment-оос Messenger чат руу шилжих урсгалыг нэмэх.

## Architecture

Би энэ ажлыг 3 тусдаа subsystem болгон хуваана. Нэгдүгээрт, product entry-г `quick add`, `bulk paste`, `excel import` гэсэн 3 оролтын сувгаар нэг validation layer-д холбож, DB write-ийг нэг shared batch API дээр төвлөрүүлнэ. Хоёрдугаарт, Messenger image upload-ийг storage + AI context болгон хувиргаж, image-aware reply болон handoff-той холбоно. Гуравдугаарт, Facebook page comment ingestion-ийг тусдаа event flow болгон авч, comment reply болон Messenger handoff боломжийг нэг conversation model дээр холбож өгнө.

Redis-ийг шинэ stateful workflow-д өргөжүүлэхгүй. Redis зөвхөн одоогийн хэрэгтэй cache/state-д, жишээ нь OAuth nonce, chat history, catalog summary зэрэг богино настай зүйлсэд үлдэнэ. Шинэ ingestion, queue, rate-limit, event state нь боломжтой газраа Postgres-backed байна.

## Tech Stack

Next.js App Router, Supabase/Postgres, existing Messenger/Facebook services, Gemini AI, existing Redis cache, storage buckets, server actions/route handlers.

---

## Scope Breakdown

### 1. Fast Product Entry

### Task 1: Shared product validation and batch write API

**Files:**
- Modify: `src/lib/import/xlsx.ts`
- Create: `src/lib/products/product-validation.ts`
- Create: `src/app/dashboard/[storeId]/products/actions.ts`
- Modify: `src/app/dashboard/[storeId]/products/import/actions.ts`
- Modify: `src/app/api/stores/[storeId]/import/[jobId]/run/route.ts`

- [ ] **Step 1: Write the failing test**

```ts
import { describe, expect, it } from "vitest";
import { validateProductRows } from "@/lib/products/product-validation";

describe("validateProductRows", () => {
  it("returns per-row errors for invalid price and empty name", () => {
    const result = validateProductRows([
      { name: "", price: -1, sku: "A1", stock_quantity: 0, category: null, description: null },
    ]);

    expect(result.validRows).toHaveLength(0);
    expect(result.errors[0].field).toBe("name");
    expect(result.errors[1].field).toBe("price");
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pnpm exec vitest run src/lib/products/product-validation.test.ts -v`
Expected: FAIL because `validateProductRows` does not exist yet.

- [ ] **Step 3: Write minimal implementation**

```ts
export function validateProductRows(rows: unknown[]) {
  return { validRows: [], errors: [] };
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pnpm exec vitest run src/lib/products/product-validation.test.ts -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add src/lib/products/product-validation.ts src/lib/products/product-validation.test.ts
```

### Task 2: Quick add and bulk paste UI

**Files:**
- Create: `src/components/dashboard/bulk-product-entry.tsx`
- Modify: `src/app/dashboard/[storeId]/products/page.tsx`
- Modify: `src/app/dashboard/[storeId]/products/actions.ts`

- [ ] **Step 1: Write the failing test**

```tsx
import { render, screen } from "@testing-library/react";
import { BulkProductEntry } from "@/components/dashboard/bulk-product-entry";

it("shows bulk paste and quick add controls", () => {
  render(<BulkProductEntry storeId="store-1" />);
  expect(screen.getByText(/bulk paste/i)).toBeInTheDocument();
  expect(screen.getByText(/quick add/i)).toBeInTheDocument();
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pnpm exec vitest run src/components/dashboard/bulk-product-entry.test.tsx -v`
Expected: FAIL because the component does not exist yet.

- [ ] **Step 3: Write minimal implementation**

```tsx
export function BulkProductEntry({ storeId }: { storeId: string }) {
  return <div>Bulk paste / Quick add</div>;
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pnpm exec vitest run src/components/dashboard/bulk-product-entry.test.tsx -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add src/components/dashboard/bulk-product-entry.tsx src/app/dashboard/[storeId]/products/page.tsx
```

### Task 3: Batch import UX polish

**Files:**
- Modify: `src/app/dashboard/[storeId]/products/import/page.tsx`
- Modify: `src/app/dashboard/[storeId]/products/import/actions.ts`
- Modify: `src/app/dashboard/[storeId]/products/import/[jobId]/page.tsx`

- [ ] **Step 1: Write the failing test**

```ts
it("surfaces validation preview before commit", async () => {
  const preview = await buildImportPreview([{ name: "", price: 10 }]);
  expect(preview.errors.length).toBeGreaterThan(0);
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pnpm exec vitest run src/app/dashboard/[storeId]/products/import/import-preview.test.ts -v`
Expected: FAIL because preview helper does not exist yet.

- [ ] **Step 3: Write minimal implementation**

```ts
export function buildImportPreview(rows: unknown[]) {
  return { rows, errors: [] };
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pnpm exec vitest run src/app/dashboard/[storeId]/products/import/import-preview.test.ts -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add src/app/dashboard/[storeId]/products/import/page.tsx src/app/dashboard/[storeId]/products/import/actions.ts src/app/dashboard/[storeId]/products/import/[jobId]/page.tsx
```

### 2. Messenger Image Support

### Task 4: Image attachment ingestion and storage

**Files:**
- Modify: `src/services/chatbot/pipeline.ts`
- Modify: `src/services/meta/types.ts`
- Create: `src/services/chatbot/image-context.ts`
- Create: `supabase/migrations/20260624000600_chat_attachments.sql`

- [ ] **Step 1: Write the failing test**

```ts
it("recognizes image attachments as image input", () => {
  const result = classifyAttachment({ type: "image", payload: {} });
  expect(result.kind).toBe("image");
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pnpm exec vitest run src/services/chatbot/image-context.test.ts -v`
Expected: FAIL because image attachment classification does not exist yet.

- [ ] **Step 3: Write minimal implementation**

```ts
export function classifyAttachment(attachment: { type?: string }) {
  return { kind: attachment.type === "image" ? "image" : "other" };
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pnpm exec vitest run src/services/chatbot/image-context.test.ts -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add src/services/chatbot/image-context.ts src/services/chatbot/pipeline.ts supabase/migrations/20260624000600_chat_attachments.sql
```

### Task 5: Image-aware AI response

**Files:**
- Create: `src/services/chatbot/vision.ts`
- Modify: `src/services/chatbot/gemini.service.ts`
- Modify: `src/services/chatbot/pipeline.ts`

- [ ] **Step 1: Write the failing test**

```ts
it("adds vision context when image attachment is present", () => {
  const prompt = buildVisionPrompt({ imageUrl: "https://example.com/x.png" });
  expect(prompt).toContain("imageUrl");
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pnpm exec vitest run src/services/chatbot/vision.test.ts -v`
Expected: FAIL because `buildVisionPrompt` does not exist yet.

- [ ] **Step 3: Write minimal implementation**

```ts
export function buildVisionPrompt(input: { imageUrl: string }) {
  return `imageUrl=${input.imageUrl}`;
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pnpm exec vitest run src/services/chatbot/vision.test.ts -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add src/services/chatbot/vision.ts src/services/chatbot/gemini.service.ts src/services/chatbot/pipeline.ts
```

### 3. Facebook Comments to Chat

### Task 6: Comment ingestion and dedup storage

**Files:**
- Create: `src/services/facebook/comments.ts`
- Create: `supabase/migrations/20260624000700_facebook_comments.sql`
- Modify: `src/app/api/webhook/route.ts`

- [ ] **Step 1: Write the failing test**

```ts
it("builds a stable comment event key", () => {
  expect(buildCommentEventKey({ pageId: "1", commentId: "2" })).toBe("comment:1:2");
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pnpm exec vitest run src/services/facebook/comments.test.ts -v`
Expected: FAIL because the helper does not exist yet.

- [ ] **Step 3: Write minimal implementation**

```ts
export function buildCommentEventKey(input: { pageId: string; commentId: string }) {
  return `comment:${input.pageId}:${input.commentId}`;
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pnpm exec vitest run src/services/facebook/comments.test.ts -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add src/services/facebook/comments.ts src/app/api/webhook/route.ts supabase/migrations/20260624000700_facebook_comments.sql
```

### Task 7: Comment reply and Messenger handoff

**Files:**
- Create: `src/services/facebook/comment-replies.ts`
- Modify: `src/services/chatbot/pipeline.ts`
- Modify: `src/app/dashboard/[storeId]/conversations/actions.ts`

- [ ] **Step 1: Write the failing test**

```ts
it("formats a reply from a comment thread into a chat handoff action", () => {
  const action = buildCommentHandoffAction({ pageId: "1", postId: "9", commentId: "2" });
  expect(action.kind).toBe("handoff");
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pnpm exec vitest run src/services/facebook/comment-replies.test.ts -v`
Expected: FAIL because helper does not exist yet.

- [ ] **Step 3: Write minimal implementation**

```ts
export function buildCommentHandoffAction(input: { pageId: string; postId: string; commentId: string }) {
  return { kind: "handoff", ...input };
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pnpm exec vitest run src/services/facebook/comment-replies.test.ts -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add src/services/facebook/comment-replies.ts src/services/chatbot/pipeline.ts src/app/dashboard/[storeId]/conversations/actions.ts
```

## Shared Guardrails

- Redis-ийг шинэ queue/counter/state-д өргөжүүлэхгүй.
- Existing Redis usage only: chat history, OAuth nonce, catalog summary cache.
- бүх шинэ tables нь Postgres-backed байна.
- comment/image/event flow бүр audit/system event-тэй холбоно.

## Spec Coverage Check

- Fast product entry: covered by Tasks 1-3
- Messenger image support: covered by Tasks 4-5
- Comment-to-chat: covered by Tasks 6-7
- Redis minimization: shared guardrails section covers it

## Open Questions to Confirm Before Implementation

- Images-ийг storage дээр хадгалсны дараа AI-д raw URL өгөх үү, эсвэл resumable upload metadata өгч vision service ашиглах уу?
- Facebook comments дээр automatic public reply default байх уу, эсвэл зөвхөн manual/owner-triggered flow байх уу?
- Bulk paste дээр delimiter нь tab-only байлгах уу, эсвэл comma/tab хоёуланг нь дэмжих үү?

## Recommended Order

1. Fast product entry
2. Messenger image support
3. Facebook comments

## Production Note

Redis-г чухал хэсэгт л үлдээх бодлого энэ план дээр хэвээр байна. Шинэ workflow state, queue, rate limit, comment/image event state-г DB дээр байлгаж, Redis-г cache зориулалттай богино настай өгөгдөлд ашиглана.
