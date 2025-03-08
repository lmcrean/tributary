# Network Calls and Data Flow

This directory contains diagrams and documentation for key network calls and data flows in the Tributary application. Each file focuses on a specific user action and includes:

1. **Overview Diagram** - A simplified sequence diagram showing the basic user-to-database flow
2. **Detailed Implementation Diagram** - A comprehensive diagram showing the actual AWS implementation with optimizations
3. **Data Payload Examples** - Request and response formats
4. **AWS Free Tier & Cost Optimizations** - Strategies to keep costs down
5. **Resilience Improvements** - How the system handles failures

## Available Flows

- [Save Job Listing](./save_job_listing.md) - How job listings are saved and processed
- [Create Keyword](./create_keyword.md) - How keywords are created and counted

## Diagram Conventions

- All diagrams use the same dark theme for consistency
- Overview diagrams only show User, Frontend, API, and Database
- Detailed diagrams show all AWS components and optimizations
- Arrows with solid lines (→) represent synchronous calls
- Arrows with dashed lines (-->) represent responses

## Common Architecture Patterns

Several architectural patterns are used consistently across the system:

1. **Caching Layer** - Reduces database load and improves response times
2. **Asynchronous Processing** - Keeps the UI responsive during heavy operations
3. **Batch Processing** - Ensures AWS free tier compatibility
4. **Progressive Loading** - Provides immediate feedback while processing continues
5. **Real-time Updates** - Uses WebSockets/SSE for pushing updates to the frontend 