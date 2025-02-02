# AI Pair Programming Guide for Next.js SaaS Project

## Tag System Reference

### Planning Tags

```xml
<plan>Define high-level implementation strategy</plan>
<subplan>Break down specific implementation steps</subplan>
<priority>Indicate task urgency and importance</priority>
<dependencies>List required dependencies or prerequisites</dependencies>
```

### Thinking Tags

```xml
<think>General thought process and reasoning</think>
<analyze>Deeper analysis of specific problems</analyze>
<evaluate>Assessment of different approaches</evaluate>
<validate>Verification of implementation correctness</validate>
<reflect>Post-implementation review and learnings</reflect>
```

### Implementation Tags

```xml
<setup>Initial configuration and environment setup</setup>
<implement>Actual code implementation details</implement>
<test>Testing strategy and implementation</test>
<refactor>Code improvement and optimization</refactor>
<document>Documentation updates and changes</document>
<server-component>Server Component implementation details</server-component>
<client-component>Client Component implementation details</client-component>
<server-action>Server Action implementation details</server-action>
<route>Route handler implementation</route>
<middleware>Middleware implementation details</middleware>
<schema>Database schema changes and migrations</schema>
<state>State management implementation</state>
<error-boundary>Error handling implementation</error-boundary>
<accessibility>Accessibility implementation details</accessibility>
<performance>Performance optimization details</performance>
```

### Communication Tags

```xml
<query>Questions about implementation details</query>
<suggest>Recommendations for improvements</suggest>
<clarify>Explanations and clarifications</clarify>
<review>Code review comments and feedback</review>
<changelog>Track changes and updates</changelog>
```

## Usage Guidelines

1. Each major task should start with a planning phase:

```xml
<plan>
  Implementing Supabase authentication
  - Set up AuthProvider
  - Configure auth middleware
  - Implement protected routes
</plan>
```

2. Before implementation, analyze requirements:

```xml
<analyze>
  Multi-tenancy requirements:
  - Subdomain handling
  - Tenant isolation
  - Data partitioning strategy
</analyze>
```

3. Document implementation decisions:

```xml
<implement>
  Creating AuthProvider component:
  - Using Supabase Auth
  - Handling session management
  - Implementing auth hooks
</implement>
```

4. Maintain changelog entries:

```xml
<changelog>
[2025-02-02]
- Added Supabase authentication setup
- Implemented tenant isolation
- Created initial database schema
</changelog>
```

## Next.js 14 Specific Guidelines

1. Server Components:

```xml
<server-component>
  Implementing data fetching dashboard:
  - Using React Suspense
  - Implementing streaming
  - Handling loading states
</server-component>
```

2. Server Actions:

```xml
<server-action>
  Implementing form submission:
  - Form validation
  - Optimistic updates
  - Error handling
</server-action>
```

3. Route Handlers:

```xml
<route>
  API implementation:
  - Edge runtime
  - Caching strategies
  - Response headers
</route>
```

4. State Management:

```xml
<state>
  Managing application state:
  - Server state
  - Form state
  - UI state
</state>
```

## Multi-tenancy Implementation Guide

1. Subdomain Routing:

```xml
<implement>
  middleware.ts configuration:
  - Tenant identification
  - Route protection
  - Authentication checks
</implement>
```

2. Database Isolation:

```xml
<schema>
  Tenant data separation:
  - Row-level security
  - Schema organization
  - Cross-tenant queries
</schema>
```

3. Tenant Management:

```xml
<server-component>
  Tenant dashboard:
  - Tenant settings
  - User management
  - Resource allocation
</server-component>
```

## Testing Strategy

1. Component Testing:

```xml
<test>
  Testing hierarchy:
  - Unit tests
  - Integration tests
  - E2E tests
  - Performance tests
</test>
```

2. Database Testing:

```xml
<test>
  Database testing:
  - Migration tests
  - Seed data
  - Cleanup strategies
</test>
```

3. Multi-tenant Testing:

```xml
<test>
  Tenant isolation:
  - Cross-tenant security
  - Tenant boundaries
  - Resource limits
</test>
```

## Performance Monitoring

1. Core Web Vitals:

```xml
<performance>
  Metrics tracking:
  - FCP
  - LCP
  - CLS
  - TTI
</performance>
```

2. Custom Metrics:

```xml
<performance>
  Business metrics:
  - API response times
  - Database query times
  - Resource utilization
</performance>
```

## Accessibility Implementation

1. ARIA Implementation:

```xml
<accessibility>
  ARIA attributes:
  - Roles
  - States
  - Properties
</accessibility>
```

2. Keyboard Navigation:

```xml
<accessibility>
  Keyboard support:
  - Focus management
  - Skip links
  - Keyboard shortcuts
</accessibility>
```

## Example Workflow

1. Starting a new feature:

```xml
<plan>Implementing Stripe subscription system</plan>
<analyze>
- Required endpoints
- Webhook handling
- Database schema updates
</analyze>
<dependencies>
- @stripe/stripe-js
- stripe webhook secret
- subscription plans defined
</dependencies>
```

2. During implementation:

```xml
<implement>
Creating subscription webhook handler:
1. Set up webhook endpoint
2. Implement signature verification
3. Handle subscription events
</implement>
<test>
- Verify webhook signature
- Test subscription creation
- Validate database updates
</test>
```

3. Post-implementation:

```xml
<reflect>
- Webhook handling improved
- Added error logging
- Consider retry mechanism
</reflect>
<changelog>
[2025-02-02] Implemented Stripe subscription system
- Added webhook handler
- Created subscription database tables
- Implemented customer portal
</changelog>
```

## Documentation Rules

1. Each component must include:

   - Purpose and functionality
   - Props interface
   - Usage examples
   - Dependencies

2. API endpoints must document:

   - Request/response schemas
   - Authentication requirements
   - Rate limiting details
   - Error handling

3. Database changes require:
   - Migration scripts
   - Schema documentation
   - Index explanations
   - Foreign key relationships

## Code Organization Guidelines

1. Feature-based structure:

```
src/
  features/
    auth/
    billing/
    tenants/
    email/
```

2. Shared components:

```
src/
  components/
    common/
    layout/
    forms/
```

3. API organization:

```
src/
  app/
    api/
      auth/
      webhooks/
      tenants/
```

## Versioning and Changelog

1. Semantic versioning:

```xml
<changelog>
v1.0.0 [2025-02-02]
BREAKING CHANGES:
- Updated to Next.js 14
- Changed authentication flow

FEATURES:
- Added multi-tenancy
- Implemented Stripe integration

FIXES:
- Resolved webhook handling
- Fixed authentication redirect
</changelog>
```

## Quality Checks

1. Before committing:

```xml
<validate>
- TypeScript compilation
- ESLint checks
- Unit tests passing
- E2E tests passing
</validate>
```

2. Code review criteria:

```xml
<review>
- Type safety
- Error handling
- Performance impact
- Security implications
- Documentation updates
</review>
```
