---
title: "Next.js Knowledge Extraction Queue"
type: resource
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - resource
  - nextjs
  - knowledge-extraction
  - documentation
  - imported
source_path: "D:/own/dusal/node_modules/next/dist/docs"
source_url: "https://github.com/vercel/next.js/tree/canary/docs"
---
# Next.js Knowledge Extraction Queue

This queue identifies local Next.js docs that are worth distilling later.

Do not convert all 421 docs into Knowledge Notes. Extract only reusable principles, patterns, decisions, and checklists.

## Candidates

### Next.js App Router Project Structure

- Target: Knowledge/Programming/Nextjs-App-Router-Project-Structure.md
- Reason: Reusable across new and existing Next.js projects.
- Sources:
  - Local: 01-app/01-getting-started/02-project-structure.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/01-getting-started/02-project-structure.mdx
  - Local: 01-app/03-api-reference/03-file-conventions/index.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/03-api-reference/03-file-conventions/index.mdx

### Server and Client Component Boundaries

- Target: Knowledge/Programming/Nextjs-Server-Client-Component-Boundaries.md
- Reason: High-leverage decision framework for App Router architecture.
- Sources:
  - Local: 01-app/01-getting-started/05-server-and-client-components.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/01-getting-started/05-server-and-client-components.mdx
  - Local: 01-app/03-api-reference/01-directives/use-client.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/03-api-reference/01-directives/use-client.mdx
  - Local: 01-app/03-api-reference/01-directives/use-server.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/03-api-reference/01-directives/use-server.mdx

### Next.js Data Fetching and Caching

- Target: Knowledge/Programming/Nextjs-Data-Fetching-and-Caching.md
- Reason: Common source of implementation mistakes and performance trade-offs.
- Sources:
  - Local: 01-app/01-getting-started/06-fetching-data.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/01-getting-started/06-fetching-data.mdx
  - Local: 01-app/01-getting-started/08-caching.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/01-getting-started/08-caching.mdx
  - Local: 01-app/01-getting-started/09-revalidating.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/01-getting-started/09-revalidating.mdx
  - Local: 01-app/02-guides/how-revalidation-works.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/how-revalidation-works.mdx

### Server Actions and Forms

- Target: Knowledge/Programming/Nextjs-Server-Actions-and-Forms.md
- Reason: Reusable mutation pattern for full-stack apps.
- Sources:
  - Local: 01-app/01-getting-started/07-mutating-data.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/01-getting-started/07-mutating-data.mdx
  - Local: 01-app/02-guides/forms.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/forms.mdx

### Route Handlers and BFF Pattern

- Target: Knowledge/Programming/Nextjs-Route-Handlers-and-BFF.md
- Reason: Useful for project architecture and API boundary decisions.
- Sources:
  - Local: 01-app/01-getting-started/15-route-handlers.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/01-getting-started/15-route-handlers.mdx
  - Local: 01-app/02-guides/backend-for-frontend.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/backend-for-frontend.mdx

### Rendering, Streaming, and Partial Prerendering

- Target: Knowledge/Programming/Nextjs-Rendering-and-Streaming.md
- Reason: Needs distilled trade-off model, not raw docs.
- Sources:
  - Local: 01-app/02-guides/rendering-philosophy.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/rendering-philosophy.mdx
  - Local: 01-app/02-guides/streaming.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/streaming.mdx
  - Local: 01-app/02-guides/ppr-platform-guide.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/ppr-platform-guide.mdx

### Next.js Security Practices

- Target: Knowledge/Programming/Nextjs-Security-Practices.md
- Reason: Reusable security checklist for production apps.
- Sources:
  - Local: 01-app/02-guides/authentication.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/authentication.mdx
  - Local: 01-app/02-guides/data-security.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/data-security.mdx
  - Local: 01-app/02-guides/content-security-policy.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/content-security-policy.mdx

### Next.js Deployment Strategy

- Target: Knowledge/Programming/Nextjs-Deployment-Strategy.md
- Reason: Helps choose between Vercel, self-hosting, and static export.
- Sources:
  - Local: 01-app/01-getting-started/17-deploying.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/01-getting-started/17-deploying.mdx
  - Local: 01-app/02-guides/deploying-to-platforms.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/deploying-to-platforms.mdx
  - Local: 01-app/02-guides/self-hosting.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/self-hosting.mdx
  - Local: 01-app/02-guides/static-exports.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/static-exports.mdx

### Next.js Testing Setup

- Target: Knowledge/Programming/Nextjs-Testing-Setup.md
- Reason: Reusable setup decision for project starts.
- Sources:
  - Local: 01-app/02-guides/testing/index.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/testing/index.mdx
  - Local: 01-app/02-guides/testing/playwright.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/testing/playwright.mdx
  - Local: 01-app/02-guides/testing/vitest.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/testing/vitest.mdx
  - Local: 01-app/02-guides/testing/jest.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/testing/jest.mdx
  - Local: 01-app/02-guides/testing/cypress.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/testing/cypress.mdx

### Next.js 16 Upgrade Notes

- Target: Resources/Nextjs-Docs/Nextjs-16-Upgrade-Reference.md
- Reason: Version-specific reference; only stable lessons should become Knowledge.
- Sources:
  - Local: 01-app/02-guides/upgrading/version-16.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/upgrading/version-16.mdx
  - Local: 01-app/01-getting-started/18-upgrading.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/01-getting-started/18-upgrading.mdx

### Next.js AI Agent Setup

- Target: Knowledge/AI/Nextjs-AI-Agent-Setup.md
- Reason: Useful for AI-assisted coding workflows.
- Sources:
  - Local: 01-app/02-guides/ai-agents.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/ai-agents.mdx
  - Local: 01-app/02-guides/mcp.md
    GitHub: https://github.com/vercel/next.js/tree/canary/docs/01-app/02-guides/mcp.mdx

## Review Rule

Promote a candidate only after it is rewritten in your own words, connected to related notes, and checked against [[05-Knowledge-Quality-Checklist]].
