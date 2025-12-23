# Feature Specification: Open Affiliate Platform

**Feature Branch**: `001-affiliate-platform`  
**Created**: 2025-12-23  
**Status**: Draft  
**Input**: User description: "I. User Roles: Seller manages products and commission, Agent gets referral links, Buyer pays and gets watermarked PDF. II. Affiliate Engine with 30-day cookie tracking. III. Python Watermarking Engine. IV. Local Payment Integration (fiuu)"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Buyer Purchases and Receives Watermarked PDF (Priority: P1)

A buyer clicks on an affiliate link, browses available digital products (PDFs), makes a payment through the local gateway (fiuu), and receives a personalized watermarked PDF with their email address embedded on every page.

**Why this priority**: This is the core revenue-generating flow. Without this working, the entire platform has no value. It demonstrates the end-to-end functionality from discovery to delivery.

**Independent Test**: Can be fully tested by: (1) Creating a test product, (2) Generating an affiliate link, (3) Processing a test payment, (4) Verifying watermarked PDF is generated and accessible. Delivers immediate value by proving the complete purchase-to-delivery pipeline works.

**Acceptance Scenarios**:

1. **Given** a buyer visits a product page with `?ref=agent123`, **When** they complete payment successfully, **Then** the system credits the sale to agent123 and delivers a watermarked PDF with the buyer's email on every page
2. **Given** a buyer visits a product page without a referral parameter, **When** they complete payment successfully, **Then** the system processes the sale without affiliate attribution and delivers a watermarked PDF
3. **Given** a buyer has an expired affiliate cookie (>30 days), **When** they complete payment, **Then** the system processes the sale without affiliate attribution
4. **Given** payment fails or is cancelled, **When** the buyer returns to the site, **Then** no PDF is generated and no commission is credited

---

### User Story 2 - Seller Creates Products and Sets Commission (Priority: P2)

A seller registers on the platform, uploads original PDF files, sets product details (title, description, price), configures commission percentage for affiliates (e.g., 50%), and publishes products to make them available for affiliate promotion.

**Why this priority**: This is the supply-side enablement. Without products, there's nothing to sell. This needs to work after the purchase flow is proven, as it provides the inventory for the core flow.

**Independent Test**: Can be tested by: (1) Registering as a seller, (2) Uploading a PDF, (3) Setting price and commission, (4) Verifying product appears in marketplace. Delivers value by enabling content creators to list their products.

**Acceptance Scenarios**:

1. **Given** a seller is authenticated, **When** they upload a PDF and set a price, **Then** the original PDF is stored securely in Supabase Storage and not publicly accessible
2. **Given** a seller configures 50% commission, **When** an affiliate sale occurs, **Then** 50% of the sale amount is credited to the referring agent's earnings
3. **Given** a seller updates product details, **When** they save changes, **Then** existing affiliate links continue to work and new sales reflect updated pricing
4. **Given** a seller deletes a product, **When** buyers access old affiliate links, **Then** the system shows a "product unavailable" message gracefully

---

### User Story 3 - Agent Joins and Generates Referral Links (Priority: P3)

An agent signs up for free, browses available products, generates unique referral links with their affiliate ID (e.g., `?ref=agent456`), shares these links through their channels, and tracks their earnings and conversion metrics in a dashboard.

**Why this priority**: This enables the affiliate marketing network, but the core purchase flow must work first. Agents need working products and proven purchase mechanisms before they'll invest time in promotion.

**Independent Test**: Can be tested by: (1) Registering as an agent, (2) Generating referral links for products, (3) Viewing earnings dashboard. Delivers value by enabling the distribution network for sellers' products.

**Acceptance Scenarios**:

1. **Given** an agent is authenticated, **When** they view product listings, **Then** they can generate a unique referral link for any published product
2. **Given** an agent shares their referral link, **When** a buyer clicks it, **Then** a 30-day cookie is set with the agent's ID
3. **Given** an affiliate sale completes, **When** the agent checks their dashboard, **Then** they see the sale, commission amount, and updated total earnings
4. **Given** an agent has no sales yet, **When** they view their dashboard, **Then** they see helpful onboarding guidance and sample links to test

---

### User Story 4 - Payment Webhook Processing (Priority: P1)

When fiuu payment gateway processes a payment (success or failure), it sends a webhook notification to the platform's webhook endpoint, which validates the signature, updates the order status in the database, and triggers PDF watermarking for successful payments.

**Why this priority**: This is critical infrastructure for the P1 buyer flow. Without reliable webhook processing, payments won't complete and PDFs won't be generated. Must be implemented alongside the purchase flow.

**Independent Test**: Can be tested by: (1) Simulating webhook payloads from fiuu, (2) Verifying order status updates, (3) Confirming PDF generation triggers. Delivers value by ensuring payment state synchronization.

**Acceptance Scenarios**:

1. **Given** a payment succeeds at fiuu, **When** the webhook arrives at `/api/webhooks/payment`, **Then** the order status updates to "paid" and PDF watermarking is triggered
2. **Given** a payment fails at fiuu, **When** the webhook arrives, **Then** the order status updates to "failed" and no PDF processing occurs
3. **Given** an invalid webhook signature, **When** a webhook request is received, **Then** the system rejects it with 401 Unauthorized and logs the security event
4. **Given** the webhook arrives before the order exists in database, **When** processing occurs, **Then** the system retries gracefully or queues the webhook for later processing

---

### User Story 5 - Affiliate Cookie Tracking via Middleware (Priority: P2)

When a buyer visits any product page with a `?ref=` parameter, Next.js middleware intercepts the request, validates the affiliate ID, sets a secure 30-day cookie, and ensures subsequent visits within 30 days maintain the attribution even if the ref parameter is absent.

**Why this priority**: This is essential infrastructure for affiliate attribution but depends on having products and agents first. It bridges the gap between agent referrals and buyer conversions.

**Independent Test**: Can be tested by: (1) Visiting pages with `?ref=test123`, (2) Verifying cookie is set, (3) Confirming attribution persists across sessions, (4) Testing cookie expiration after 30 days. Delivers value by ensuring accurate commission attribution.

**Acceptance Scenarios**:

1. **Given** a buyer visits `/product/123?ref=agent789`, **When** the page loads, **Then** a secure HttpOnly cookie is set with agent789 for 30 days
2. **Given** a buyer already has an affiliate cookie, **When** they visit a new link with a different ref parameter, **Then** the cookie updates to the latest affiliate ID
3. **Given** a buyer has an affiliate cookie set, **When** they return directly to the site without a ref parameter, **Then** the cookie persists and attribution is maintained
4. **Given** an invalid or non-existent affiliate ID in the ref parameter, **When** the middleware processes it, **Then** no cookie is set and the request continues normally

---

### Edge Cases

- What happens when a buyer clicks multiple different affiliate links before purchasing? (Last-click attribution wins per US5)
- What happens when the Python watermarking service is down during a purchase? (Order marked as "pending_processing", retry mechanism attempts watermarking)
- What happens when a seller uploads a corrupted or password-protected PDF? (Upload validation rejects invalid PDFs before storage)
- What happens when fiuu webhook is delivered multiple times (idempotency)? (Order status updates are idempotent, duplicate webhooks don't cause double-processing)
- What happens when a buyer tries to access someone else's watermarked PDF? (Supabase RLS policies prevent unauthorized access)
- What happens when an agent tries to use their own referral link to buy products? (System should detect and either prevent commission or flag for review)
- What happens when commission percentage changes after an affiliate link was generated? (Commission at time of sale is used, stored in orders table)
- What happens when a watermarked PDF generation takes longer than expected? (Buyer receives email notification when PDF is ready, order status shows "processing")

## Requirements *(mandatory)*

### Functional Requirements

**User Management & Roles**:
- **FR-001**: System MUST support three distinct user roles: Seller, Agent, and Buyer
- **FR-002**: System MUST use Supabase Auth for email/password authentication
- **FR-003**: Users MUST be able to register with email verification
- **FR-004**: Sellers and Agents MUST have role-specific dashboards after authentication
- **FR-005**: System MUST allow users to have multiple roles (e.g., a user can be both Seller and Agent)

**Product Management (Seller)**:
- **FR-006**: Sellers MUST be able to upload PDF files up to 50MB in size
- **FR-007**: System MUST store original PDFs in Supabase Storage with restricted access
- **FR-008**: Sellers MUST be able to set product title, description, price (in MYR), and commission percentage
- **FR-009**: System MUST validate that commission percentage is between 0% and 100%
- **FR-010**: Sellers MUST be able to view their sales history and total revenue
- **FR-011**: Sellers MUST be able to soft-delete (unpublish) products
- **FR-012**: System MUST prevent sellers from deleting products with pending orders

**Affiliate System (Agent)**:
- **FR-013**: Agents MUST be able to register for free without payment information
- **FR-014**: System MUST generate unique referral links with format: `{base_url}/product/{id}?ref={agent_id}`
- **FR-015**: System MUST track affiliate cookie for 30 days from last click
- **FR-016**: System MUST use last-click attribution model (most recent ref parameter wins)
- **FR-017**: Agents MUST be able to view their total earnings, pending commissions, and conversion metrics
- **FR-018**: System MUST calculate commission based on percentage set at time of sale
- **FR-019**: System MUST credit commission to agent's profile in Supabase immediately upon payment confirmation

**Purchase & Payment (Buyer)**:
- **FR-020**: Buyers MUST be able to browse products without authentication
- **FR-021**: Buyers MUST authenticate before checkout
- **FR-022**: System MUST integrate with fiuu payment gateway for MYR transactions
- **FR-023**: System MUST support credit card, debit card, and FPX (Malaysian online banking) payment methods
- **FR-024**: System MUST provide a webhook endpoint at `/api/webhooks/payment` for payment notifications
- **FR-025**: System MUST validate webhook signatures from fiuu to prevent fraud
- **FR-026**: System MUST update order status to "paid", "failed", or "pending" based on webhook data
- **FR-027**: System MUST handle idempotent webhook processing (duplicate webhooks don't cause issues)

**PDF Watermarking**:
- **FR-028**: System MUST trigger Python watermarking service when order status becomes "paid"
- **FR-029**: Python service MUST download original PDF from Supabase Storage using service role key
- **FR-030**: Python service MUST apply watermark text "Sold to: {buyer_email}" on every page
- **FR-031**: Watermark MUST be positioned consistently (e.g., bottom-right corner, semi-transparent)
- **FR-032**: Python service MUST upload watermarked PDF to separate "Watermarked" bucket in Supabase Storage
- **FR-033**: System MUST generate time-limited signed URL for buyer to download watermarked PDF
- **FR-034**: System MUST implement retry mechanism for failed watermarking jobs (max 3 retries)
- **FR-035**: System MUST notify buyer via email when watermarked PDF is ready

**Middleware & Cookie Tracking**:
- **FR-036**: Next.js middleware MUST intercept all requests with `?ref=` parameter
- **FR-037**: Middleware MUST validate affiliate ID exists in database before setting cookie
- **FR-038**: Middleware MUST set secure, HttpOnly cookie with 30-day expiration
- **FR-039**: Middleware MUST update cookie if user clicks new affiliate link
- **FR-040**: System MUST read affiliate cookie during checkout to attribute sale

**Data Security & Access Control**:
- **FR-041**: Original PDFs MUST be stored with private access (only seller and system can access)
- **FR-042**: Watermarked PDFs MUST be accessible only to the buyer who purchased them (via RLS policies)
- **FR-043**: System MUST use Supabase Row-Level Security for all user-facing tables
- **FR-044**: Payment webhook endpoint MUST validate request signatures
- **FR-045**: Inter-service communication (Next.js ↔ Python) MUST use service role keys, not user JWTs

### Key Entities

- **User**: Represents all platform users with fields: id, email, role (enum: seller/agent/buyer), created_at, updated_at. A user can have multiple roles through a many-to-many relationship.
- **Product**: Represents digital PDFs for sale with fields: id, seller_id (FK to User), title, description, price, commission_percentage, original_pdf_url, status (published/unpublished), created_at, updated_at, deleted_at
- **Agent Profile**: Extends User with fields: user_id (FK), total_earnings, pending_commission, referral_code (unique), joined_at
- **Order**: Represents a purchase transaction with fields: id, buyer_id (FK to User), product_id (FK to Product), affiliate_id (nullable FK to Agent Profile), amount, commission_amount, status (pending/paid/failed/processing_pdf), payment_gateway_id, watermarked_pdf_url (nullable), created_at, updated_at
- **Affiliate Click**: Tracks affiliate link clicks with fields: id, agent_id (FK to Agent Profile), product_id (FK to Product), clicked_at, ip_address, user_agent. Used for analytics and conversion tracking.
- **Commission**: Represents commission payments with fields: id, order_id (FK to Order), agent_id (FK to Agent Profile), amount, status (pending/paid), paid_at

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Buyers can complete the entire purchase journey (product selection → payment → PDF delivery) in under 5 minutes
- **SC-002**: PDF watermarking completes within 60 seconds for 95% of orders (PDFs up to 50MB, 100 pages)
- **SC-003**: System maintains 99.5% uptime for payment webhook processing
- **SC-004**: Affiliate attribution accuracy is 100% (every sale with valid cookie is correctly attributed)
- **SC-005**: 90% of buyers successfully download their watermarked PDF on first attempt without support intervention
- **SC-006**: Zero unauthorized access to original or watermarked PDFs (validated through security audit)
- **SC-007**: Payment processing supports 100 concurrent checkout sessions without performance degradation
- **SC-008**: Agents can generate and share referral links within 30 seconds of product discovery
- **SC-009**: Sellers can publish a new product (upload + configure) in under 3 minutes
- **SC-010**: System handles duplicate webhook deliveries with 100% idempotency (no double-processing of payments)

## Assumptions *(optional)*

- fiuu payment gateway documentation and API credentials will be provided during implementation
- Buyers have valid email addresses for receiving watermarked PDF delivery notifications
- PDF files are standard-compliant (not corrupted, not password-protected) as upload validation will reject invalid files
- Commission payouts to agents will be handled in a future feature (this feature only tracks earnings)
- MYR (Malaysian Ringgit) is the only supported currency initially
- All products are digital PDFs; other file formats are not supported in this version
- Sellers are responsible for ensuring they have rights to sell the PDFs they upload
- Watermark placement (bottom-right corner) is acceptable for all PDF types and doesn't interfere with content
- Python watermarking service will run on the same infrastructure as Next.js (Docker Compose locally, separate container in production)
- Supabase will handle 1000+ concurrent database operations without custom scaling
- Buyers will not attempt to remove watermarks (enforcement mechanisms are out of scope)

## Out of Scope *(optional)*

- Multi-language support (English only for MVP)
- Mobile apps (web-responsive only)
- Agent payout/withdrawal functionality (tracking only)
- Product categories and search filters (future enhancement)
- Product reviews and ratings
- Multiple currency support
- Subscription-based products (one-time purchases only)
- Batch upload for sellers (one product at a time)
- Advanced analytics dashboard (basic metrics only)
- Refund processing
- Discount codes and promotions
- Guest checkout (authentication required)
- Social media OAuth login (email/password only)
- PDF preview before purchase
- Seller verification/KYC process
