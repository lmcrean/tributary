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

## MVP Focus

The current diagrams focus on the core MVP functionality:
- Keyword tracking and counting
- Job listing management
- Real-time updates of keyword mentions

Status categorization (have/learning/need) and color coding have been intentionally omitted from the MVP documentation to maintain focus on the essential counting feature.

## Diagram Conventions

- All diagrams use the same dark theme for consistency
- Overview diagrams only show User, Frontend, API, and Database
- Detailed diagrams show all AWS components and optimizations
- Arrows with solid lines (→) represent synchronous calls
- Arrows with dashed lines (-->) represent responses

## Common Architecture Patterns

Both operations follow these consistent architectural patterns:

1. **Caching Layer**:
   - All user data access first checks cache
   - Cache misses automatically populate cache
   - Both API and Workers utilize the cache

2. **Asynchronous Processing**:
   - Heavy computations are offloaded to background workers
   - SQS queues decouple the operations
   - Initial quick responses followed by push notifications

3. **Batch Processing & Pagination**:
   - Both operations process data in batches of 25
   - Save Job Listing - Processes keywords in batches
   - Create Keyword - Processes job listings in batches
   - Ensures operations stay within AWS free tier limits

4. **Progressive Loading Pattern**:
   - All operations return immediate success response
   - Background processing continues asynchronously
   - WebSocket/SSE used to push updates to the frontend

5. **Resilience First Design**:
   - Core functionalities (saving/creating) work even if background processing fails
   - Cache fallbacks ensure system degradation is graceful
   - Consistent messaging scheme for all WebSocket/SSE communications

## Alignment with System Design

These network calls directly support the core ERD relationships:
- User-JobListing: Save Job Listing flow
- User-UserKeyword: Create Keyword flow
- JobListing-UserKeyword: Both flows handle the many-to-many relationship by calculating mentions 