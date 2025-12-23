<!--
SYNC IMPACT REPORT
==================
Version: 1.0.0 → 1.1.0 (Technical Stack Specification)
Last amended: 2025-12-23

CHANGES SUMMARY:
- Added comprehensive Technical Stack section with Next.js 15 and Python 3.11 specifications
- Documented Docker Compose architecture with three services (web, worker, db)
- Specified Supabase as Backend-as-a-Service (Auth, PostgreSQL, Storage)
- Defined inter-service communication patterns (Storage bucket + internal REST API)
- Added technology-specific constraints for PDF processing and affiliate tracking
- Updated Repository Structure to reflect actual workspaces: web/, worker/, docs/, database/

SECTIONS MODIFIED:
- Monorepo Constraints > Repository Structure (UPDATED)
- Monorepo Constraints > Technical Stack (NEW - materially expanded guidance)

TEMPLATES STATUS:
✅ plan-template.md - Compatible, web application structure applies
✅ spec-template.md - No changes required
✅ tasks-template.md - No changes required

FOLLOW-UP ACTIONS:
- None required at this time

JUSTIFICATION FOR VERSION 1.1.0 (MINOR BUMP):
This amendment materially expands guidance by adding comprehensive technical stack
specifications, Docker Compose architecture details, and technology-specific
constraints. Per governance versioning policy, this constitutes a MINOR version
change as it adds new sections and materially expands guidance without removing
or redefining existing principles.

PREVIOUS SYNC REPORT (1.0.0):
Version: 0.0.0 → 1.0.0 (Initial Release)
Constitution created: 2025-12-23
- Initial constitution establishment
- Defined 5 core principles for monorepo architecture
-->

# Open Affiliate Constitution

## Core Principles

### I. Workspace Isolation

**Declaration**: Each workspace (documentation, frontend, backend, database) MUST be independently buildable, testable, and deployable with clearly defined boundaries.

**Rules**:
- Each workspace maintains its own dependencies, configuration, and build process
- No workspace may directly import from another workspace's internal modules
- Shared code MUST be extracted into a dedicated shared library workspace
- Each workspace MUST have its own test suite that runs independently
- Build failures in one workspace MUST NOT prevent building other workspaces

**Rationale**: Workspace isolation enables parallel development, reduces coupling, allows independent scaling, and simplifies maintenance. Teams can work on different parts of the system without coordination overhead or unexpected breakage.

### II. Contract-Driven Integration

**Declaration**: All inter-workspace communication MUST occur through explicitly defined, versioned contracts (APIs, database schemas, event schemas).

**Rules**:
- API contracts MUST be documented in OpenAPI/GraphQL schema format before implementation
- Database schema changes MUST follow migration-first approach with backward compatibility
- Breaking changes to contracts require MAJOR version bump and migration path documentation
- Contract changes MUST be reviewed and approved before implementation begins
- All contracts MUST include validation rules and error handling specifications

**Rationale**: Explicit contracts create clear boundaries, enable independent evolution of workspaces, facilitate testing through mocking, and provide documentation that never drifts from implementation.

### III. Test-First Development (NON-NEGOTIABLE)

**Declaration**: Tests MUST be written and approved before implementation begins. The Red-Green-Refactor cycle is mandatory.

**Rules**:
- Write test cases that capture acceptance criteria → Get user/reviewer approval → Verify tests fail → Implement until tests pass
- Each user story MUST have corresponding test scenarios before coding
- Contract tests MUST exist for all API endpoints and database schemas
- Integration tests MUST cover critical inter-workspace communication paths
- Test coverage MUST NOT decrease with new changes

**Rationale**: Test-first development ensures we build what was actually requested, provides regression protection, serves as executable documentation, and enables confident refactoring. It prevents scope creep and gold plating.

### IV. Independent Deployability

**Declaration**: Each workspace MUST be deployable independently without requiring coordinated deployment of other workspaces.

**Rules**:
- Frontend MUST handle backend API unavailability gracefully
- Backend MUST support multiple frontend versions through API versioning
- Database migrations MUST be backward compatible with previous application version
- Feature flags MUST be used for features requiring coordination across workspaces
- Each workspace MUST maintain its own deployment pipeline and rollback capability

**Rationale**: Independent deployability reduces deployment risk, enables faster iteration, allows rolling deployments, and prevents system-wide outages from single-workspace issues.

### V. Observability & Monitoring

**Declaration**: All workspaces MUST implement comprehensive logging, metrics, and tracing to enable debugging and performance monitoring in production.

**Rules**:
- Structured logging (JSON format) MUST be used with consistent fields across workspaces
- All API requests MUST include correlation IDs that flow through all workspaces
- Critical operations MUST emit metrics (latency, error rate, throughput)
- Error scenarios MUST be logged with sufficient context for diagnosis
- Health check endpoints MUST be implemented for all services

**Rationale**: In a distributed monorepo system, observability is critical for understanding system behavior, diagnosing issues, and measuring performance. Consistent practices across workspaces enable effective system-wide monitoring.

## Monorepo Constraints

### Repository Structure

The repository MUST maintain the following workspace organization:

```
docs/              # Documentation workspace (Markdown, diagrams, API specs, guides)
web/               # Next.js 15 frontend and API routes workspace
worker/            # Python 3.11 PDF processing engine workspace
database/          # Supabase migrations and schema definitions
docker-compose.yml # Local development orchestration
```

**Workspace Responsibilities**:

**web/** - Next.js 15 Application
- Frontend UI components and pages
- API routes for client-server communication
- Server-side rendering and static generation
- Authentication integration with Supabase
- Affiliate tracking link generation and validation

**worker/** - Python 3.11 PDF Engine
- PDF watermarking using pypdf or reportlab
- PDF generation and manipulation
- Internal REST API for web service communication
- Supabase Storage integration for PDF file handling
- Background job processing

**database/** - Supabase & PostgreSQL
- Database schema definitions
- Migration scripts (sequential, timestamped)
- Supabase configuration (auth rules, storage policies, RLS policies)
- Seed data for development/testing

**docs/** - Project Documentation
- Architecture diagrams and decision records
- API documentation (OpenAPI specs)
- Setup and deployment guides
- Development workflow documentation

**Requirements**:
- Each workspace MUST have its own README.md explaining setup, dependencies, and build process
- Root-level tooling configuration (ESLint, Prettier, TypeScript config) applies to applicable workspaces
- Workspace-specific configuration overrides root configuration when needed
- Docker Compose MUST be used for local development environment consistency

### Technical Stack

**Frontend & API Layer**:
- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript 5.x
- **Runtime**: Node.js 20.x LTS
- **Package Manager**: npm or pnpm
- **Key Libraries**: React 19, Supabase JS Client
- **Location**: `web/` workspace

**PDF Processing Engine**:
- **Language**: Python 3.11+
- **Framework**: FastAPI or Flask for internal API
- **PDF Libraries**: pypdf (for manipulation), reportlab (for generation)
- **Package Manager**: pip with requirements.txt or Poetry
- **Location**: `worker/` workspace

**Backend-as-a-Service**:
- **Platform**: Supabase
- **Database**: PostgreSQL 15+ (via Supabase)
- **Authentication**: Supabase Auth (email/password, OAuth providers)
- **Storage**: Supabase Storage (for PDF files, user uploads)
- **Real-time**: Supabase Realtime (optional, for live updates)

**Infrastructure**:
- **Orchestration**: Docker Compose (local development)
- **Services**:
  - `web`: Next.js application (port 3000)
  - `worker`: Python PDF engine (internal port 8000)
  - `db`: Supabase local instance or remote connection
- **Production**: Vercel (Next.js) + Docker container (Python worker) + Supabase Cloud

### Technology-Specific Constraints

**Affiliate Tracking**:
- Tracking links MUST use the query parameter format: `?ref={affiliateId}`
- Attribution data MUST be stored in Supabase `orders` table with affiliate_id foreign key
- Affiliate IDs MUST be validated before order attribution
- Cookie-based tracking SHOULD be implemented for attribution persistence (7-30 days)
- Invalid or expired affiliate links MUST gracefully degrade (process order without attribution)

**PDF Security & Processing**:
- Dynamic watermarks MUST include: affiliate ID, purchase date, customer email hash
- Watermarks MUST be applied server-side in Python worker (never client-side)
- PDF files MUST be stored in Supabase Storage with Row-Level Security (RLS) policies
- Original PDFs MUST be kept separate from watermarked PDFs (different storage buckets)
- PDF processing jobs MUST be idempotent (safe to retry)

**Inter-Service Communication**:
- **Primary Method**: Supabase Storage bucket as shared file system
  - Next.js writes job requests to Storage (e.g., `/jobs/{uuid}.json`)
  - Python worker polls or listens for new job files
  - Worker writes results back to Storage (e.g., `/results/{uuid}.pdf`)
- **Alternative Method**: Internal REST API
  - Python worker exposes internal HTTP endpoints (not publicly accessible)
  - Next.js calls worker API with job details
  - Worker responds with job ID, Next.js polls for completion
- Communication pattern MUST be documented in contracts/ directory
- Authentication between services MUST use service role keys (not user JWTs)

**Database Schema Requirements**:
- All tables MUST have `created_at` and `updated_at` timestamps
- Soft deletes MUST be implemented using `deleted_at` column (not hard deletes)
- Row-Level Security (RLS) policies MUST be enabled on all user-facing tables
- Foreign key constraints MUST be defined with appropriate ON DELETE behavior
- Indexes MUST be created for all foreign keys and frequently queried columns

### Versioning Strategy

- **Workspace Versioning**: Each workspace maintains independent semantic versioning (MAJOR.MINOR.PATCH)
- **API Versioning**: Backend APIs use URI versioning (/v1/, /v2/) for breaking changes
- **Database Versioning**: Sequential migration numbers with timestamp prefixes
- **Release Coordination**: Cross-workspace releases documented in CHANGELOG.md with dependency matrix

## Development Workflow

### Feature Development Process

1. **Specification**: Feature spec written following spec-template.md with user stories prioritized
2. **Planning**: Implementation plan created following plan-template.md with workspace impact analysis
3. **Contract Definition**: API/schema contracts defined and reviewed before implementation
4. **Test Creation**: Tests written per Test-First Development principle
5. **Implementation**: Code written following tasks.md with regular commits
6. **Integration Validation**: Cross-workspace integration tested in staging environment
7. **Documentation**: User-facing documentation updated in docs/ workspace
8. **Deployment**: Independent workspace deployments with monitoring validation

### Code Review Requirements

- All code changes MUST be reviewed by at least one other developer
- Constitution compliance MUST be verified during review
- Breaking changes MUST be explicitly called out and justified
- Test coverage MUST be maintained or improved
- Documentation MUST be updated to reflect changes

## Governance

**Authority**: This constitution supersedes all other development practices and guidelines. Any conflict between this document and other documentation MUST be resolved in favor of the constitution.

**Amendment Process**:
- Amendments MUST be proposed through documented RFC (Request for Comments) process
- Proposed changes MUST include justification and impact analysis
- Amendments require approval from project maintainers
- Amendments MUST be versioned and include migration guidance if needed

**Compliance**:
- All pull requests and code reviews MUST verify compliance with constitutional principles
- Violations MUST be documented in Complexity Tracking section of plan.md with justification
- Repeated violations without justification MAY result in PR rejection
- Constitution compliance is measured during retrospectives and feature reviews

**Versioning Policy**:
- MAJOR version: Backward incompatible governance changes, principle removals or redefinitions
- MINOR version: New principles added, materially expanded guidance, new sections
- PATCH version: Clarifications, wording improvements, typo fixes, non-semantic refinements

**Living Document**: This constitution is reviewed quarterly and updated as the project evolves. Development guidance for AI agents working on this project is maintained in `.github/prompts/` directory.

**Version**: 1.1.0 | **Ratified**: 2025-12-23 | **Last Amended**: 2025-12-23
