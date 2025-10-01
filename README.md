# StreamSave

**A responsive, secure, and accessible web application for authorized video metadata viewing and downloading.**

---

## Project Brief

Build a responsive, secure, and accessible web application called **StreamSave** (placeholder name) that lets authenticated users view metadata and download videos they are authorized to download from supported platforms. The product must have a modern, mobile-first UI/UX, high performance, robust error handling, and automated tests. The app should be deployable to a cloud provider and be production-ready.

---

## Goal & Success Criteria

### Acceptance Tests

- ✅ User can sign up / sign in securely (OAuth + email) and view a dashboard showing authorized downloadable videos
- ✅ User can add a video URL or connect their YouTube account via OAuth to list their own uploaded videos and download them (if permitted)
- ✅ Download process must not circumvent platform protections — only allow downloads for content where explicit permission exists
- ✅ **UX:** Pages load within 1s on 4G; responsive layout across breakpoints; WCAG AA accessibility
- ✅ **Security:** No leaking of credentials, all sensitive actions logged, rate limits in place
- ✅ CI pipeline with automated unit, integration, and E2E tests; staging and production deployments available
- ✅ Clear documentation and a short demo video showing flows

---

## Target Users & User Stories

### Target Users

1. **Content creators:** "As a creator, I want to download my own uploads in different resolutions so I can back them up."
2. **Licensed distributors:** "As a licensee, I want to download videos I have the rights to for offline display."
3. **Researchers/Students:** "As a researcher with permission, I want to archive an approved lecture video."

### Sample User Stories

- As a logged-in user who connected their YouTube account, I can see a list of my uploaded videos with metadata and a download button for each video I own
- As a user who uploads a file to the app, I can download transcoded copies (mp4 720p, 480p, 360p)
- As an admin, I can view audit logs of downloads and revoke tokens

---

## Functional Requirements

### Core Features

- **Secure auth:** Email/password (with verification) + OAuth (Google/YouTube) login for listing owned uploads
- **Video listing:** For OAuth-connected YouTube accounts show title, duration, upload date, thumbnails, available formats (based on metadata)
- **Download:** Allow download only for user-owned or explicitly permitted content. For user-owned content support multiple formats/resolutions
- **Upload & conversion:** Let users upload their content and request transcoding to common formats
- **Job queue:** Use background workers for processing/conversion and notify UI with progress
- **Logs & analytics:** Track downloads and provide basic analytics per user
- **Rate limiting + abuse detection**

### Advanced Features

- Batch download (zipped) for multiple authorized videos
- Scheduled backups for a user's uploads (export to S3 / cloud storage)
- Webhooks for processing completion and notifying external systems
- Client-side resume support for large downloads (range requests)
- Internationalization (i18n) support
- Fine-grained admin console for managing permissions and review requests

---

## Non-Functional Requirements

- **Performance:** 95% of API calls < 300ms; downloads use efficient streaming, support range requests
- **Scalability:** Stateless API servers, horizontally scalable workers and object storage
- **Availability:** Deploy across at least two availability zones, graceful degradation
- **Security:** OAuth tokens stored encrypted; least privilege IAM; input validation; CSP headers; rate limiting
- **Accessibility:** WCAG AA compliance, keyboard navigable components, semantic HTML, ARIA labels
- **Privacy/compliance:** GDPR-compliant data handling, delete-on-request, audit logs retention policy

---

## UX / UI Direction

### Look & Feel

- **Design:** Clean, minimal, glassmorphism accents, rounded cards, large thumbnails
- **Interactions:** Micro-interactions for progress, motion subtle and purposeful
- **Brand Palette (example):**
  - Primary: `#0B57D0` (deep blue)
  - Accent: `#00C2A8` (teal)
  - Neutral: soft grays
- **Themes:** Light & dark mode support

### Layout

- **Desktop:** Left vertical nav
- **Mobile:** Bottom nav
- **Dashboard Cards:** "Connected Accounts", "My Videos", "Recent Downloads", "Upload + Convert"

### Key Components

- Video card (thumbnail, title, metadata, action menu)
- Connect account modal (OAuth)
- Download modal (choose resolution/format)
- Progress toast
- Activity feed

### Microcopy

- Friendly, explicit permission reminders
- Download size estimate
- Copyright warning before any download

---

## Tech Stack

### Frontend

- **Framework:** React + TypeScript
- **Build Tool:** Vite
- **Styling:** TailwindCSS (or Tailwind + shadcn/ui components)
- **State Management:** React Query for data caching
- **Routing:** React Router
- **Animations:** Framer Motion

### Backend

- **Runtime:** Node.js (TypeScript) with Fastify or NestJS
- **Alternative:** Python (FastAPI)

### Worker/Queue

- Redis + BullMQ or RabbitMQ for background jobs

### Storage

- S3-compatible object storage (AWS S3, DigitalOcean Spaces)

### Database

- PostgreSQL (Prisma or TypeORM)

### Auth

- OAuth 2.0 (Google) for YouTube integration
- JWT for session management
- Secure provider for secrets (AWS Secrets Manager / Vault)

### Transcoding

- FFmpeg for user-owned uploads and conversions

### CI/CD

- GitHub Actions (test, lint, build)
- Deploy frontend to Vercel
- Deploy backend + workers to AWS/GCP/Azure

### Monitoring

- Sentry for errors
- Prometheus + Grafana for metrics
- CloudWatch for logs (or equivalent)

---

## Project Structure

```
StreamSave/
├── frontend/           # React + TypeScript frontend
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.ts
├── backend/            # Node.js/TypeScript backend API
│   ├── src/
│   ├── package.json
│   └── tsconfig.json
├── worker/             # Background job processor
│   ├── src/
│   ├── package.json
│   └── tsconfig.json
├── docs/               # Documentation
│   ├── API.md
│   ├── DEPLOYMENT.md
│   └── ARCHITECTURE.md
├── figma/              # Design files and exports
├── .github/
│   └── workflows/      # CI/CD pipelines
├── .gitignore
├── LICENSE             # MIT License
└── README.md           # This file
```

---

## API & Integration Notes

### YouTube OAuth Flow

- Request scopes strictly needed to list/manage the user's own videos (YouTube Data API scopes)
- Use server-side token exchange and store refresh tokens encrypted
- Provide a way to revoke tokens in-app

### Metadata

- Use YouTube Data API to list user uploads and metadata
- Only allow downloading when ownership is confirmed by OAuth

### Download Storage/Streaming

- For allowed downloads, create a signed temporary URL pointing to a stored processed file in S3
- Stream from worker to client with proper content-disposition headers
- Support range requests for resume

### Webhook & Jobs

- When a user requests a conversion/download, create a job in the queue
- Worker downloads/transcodes only user-uploaded files or files the user explicitly provided and has rights to
- Worker stores final artifacts in object storage and the API returns a signed URL

---

## UI Flows

### Onboarding

1. Sign up → Connect Google/YouTube (optional) → Grant scopes → Dashboard

### List Videos

2. If connected, show list of owned uploads; if not, users can upload videos
3. Each video has metadata and actions

### Request Download

4. Click Download → choose format/resolution → confirm ownership/consent → job queued → progress shown → download available (signed URL)

### Batch Download

5. Select multiple authorized items → request zip → background job → notification when ready

---

## Testing & QA

- **Unit tests** for components and backend modules
- **Integration tests** for auth flow, job queue, and storage interactions
- **E2E tests** for main user flows (Cypress/Playwright)
- **Accessibility tests** (axe-core)
- **Load tests** for worker queue and download endpoints (k6 or Artillery)
- **Security tests:** automated SAST, dependency scanning, token leakage checks

---

## Milestones

### Week 1
Project setup, CI, basic auth, dashboard skeleton (Figma + frontend scaffold)

### Week 2
OAuth integration, list user uploads (metadata), upload endpoint for user files

### Week 3
Job queue, worker implementation, FFmpeg-based conversion for uploads, download flow (signed URLs)

### Week 4
Admin panel, analytics, accessibility fixes, E2E tests, staging deployment

### Week 5
Final QA, documentation, production deployment, demo video, handover

---

## Deliverables

Each milestone delivered with:
- Tests
- Documentation
- Migration scripts
- Changelog

---

## Acceptance Criteria / QA Checklist

- [ ] OAuth flow securely implemented and tested
- [ ] Downloads only work for user-owned or explicitly permitted content
- [ ] All pages responsive, accessible (WCAG AA)
- [ ] Background jobs and worker recover from failures
- [ ] Signed URLs expire; tokens encrypted
- [ ] E2E tests pass in CI
- [ ] Security scan shows no critical issues

---

## Legal & Ethical Notes

⚠️ **Important:**

- Provide an explicit mechanism for users to prove/attest ownership (e.g., OAuth listing, or an upload + token verification)
- On the UI, before any download, show a short tooltip/modal reminding users to confirm they own the content or have rights; require a checkbox confirming consent. Log the consent.
- Keep a mechanism for takedown requests and retention policies
- Consult legal counsel if adding features that might bear copyright risk in target jurisdictions

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Contributing

Contributions are welcome! Please read our contributing guidelines and code of conduct before submitting pull requests.

---

## Contact

For questions or support, please open an issue on GitHub.

---

**Built with ❤️ for secure, ethical content management**
