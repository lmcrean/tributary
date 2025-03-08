# Create Keyword

This document outlines the network calls and data flow for creating a keyword in the Tributary application.

## Overview Diagram

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryTextColor': '#ffffff', 'primaryColor': '#ffffff', 'noteBkgColor': '#2b2b2b', 'noteTextColor': '#ffffff', 'activationBorderColor': '#ffffff', 'signalColor': '#ffffff' }}}%%
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Database

    User->>Frontend: Clicks "+ Add Custom Keyword" button
    Frontend->>Frontend: Displays keyword creation modal
    User->>Frontend: Enters keyword name
    User->>Frontend: Clicks "Save Keyword" button
    Frontend->>API: POST /api/keywords/create
    API->>Database: Check if keyword already exists for user
    Database-->>API: Return existence check result
    
    alt Keyword already exists
        API-->>Frontend: Return error (duplicate keyword)
        Frontend-->>User: Show error message
    else Keyword doesn't exist
        API->>Database: Save new keyword
        API->>Database: Calculate keyword mentions across job listings
        Database-->>API: Return updated keyword with counts
        API-->>Frontend: Return success + keyword data
        Frontend-->>User: Show "Keyword created" confirmation
        Frontend->>Frontend: Add keyword to tracker UI
    end
```

## Detailed Implementation Diagram

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryTextColor': '#ffffff', 'primaryColor': '#ffffff', 'noteBkgColor': '#2b2b2b', 'noteTextColor': '#ffffff', 'activationBorderColor': '#ffffff', 'signalColor': '#ffffff' }}}%%
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Cache
    participant Queue
    participant Worker
    participant Database

    User->>Frontend: Clicks "+ Add Custom Keyword" button
    Frontend->>Frontend: Displays keyword creation modal
    User->>Frontend: Enters keyword name
    User->>Frontend: Clicks "Save Keyword" button
    Frontend->>API: POST /api/keywords/create
    
    API->>Cache: Check if keyword exists for user
    Cache-->>API: Return cache hit/miss
    
    alt Cache Miss
        API->>Database: Check if keyword already exists
        Database-->>API: Return existence check result
        API->>Cache: Update cache with result
    end
    
    alt Keyword already exists
        API-->>Frontend: Return error (duplicate keyword)
        Frontend-->>User: Show error message
    else Keyword doesn't exist
        API->>Database: Save new keyword
        Database-->>API: Confirm save
        API-->>Frontend: Return success (keyword created)
        Frontend-->>User: Show "Keyword created" confirmation
        Frontend->>Frontend: Add keyword to tracker UI with "calculating..." status
        
        API->>Queue: Enqueue keyword counting task
        Note over Queue: Decouples creation from heavy computation
        
        Queue->>Worker: Process keyword counting task
        
        Worker->>Cache: Check for user's job listings data
        Cache-->>Worker: Return cache hit/miss
        
        alt Cache Miss
            Worker->>Database: Get pagination metadata for job listings
            Database-->>Worker: Return pagination data
            Worker->>Cache: Update cache with pagination data
        end
        
        Worker->>Database: Get user's job listings (paginated, batch 1)
        Database-->>Worker: Return batch 1 job listings
        Worker->>Worker: Calculate keyword mentions for batch 1
        
        loop Until all job listings processed (batch size: 25)
            Worker->>Database: Get next batch of job listings
            Database-->>Worker: Return next batch
            Worker->>Worker: Calculate keyword mentions for batch
            Note over Worker: Stays within AWS free tier limits
        end
        
        Worker->>Database: Update keyword totalMentions (atomic operation)
        Database-->>Worker: Confirm update
        
        Worker->>Frontend: Push update via WebSocket/SSE
        Frontend->>Frontend: Update keyword count in UI
    end
```

## Data Payload Examples

```json
// POST /api/keywords/create request
{
  "name": "TypeScript"
}

// Initial success response (immediate)
{
  "success": true,
  "keyword": {
    "keywordId": 123,
    "name": "TypeScript",
    "isCustom": true,
    "totalMentions": "calculating..."
  }
}

// WebSocket/SSE update (async)
{
  "type": "KEYWORD_COUNT_UPDATED",
  "keywordId": 123,
  "totalMentions": 3
}
```

## AWS Free Tier & Cost Optimizations

The detailed implementation diagram incorporates several optimizations to ensure the application stays within AWS free tier limits and minimizes costs:

1. **Caching Layer**:
   - Uses ElastiCache (Redis) to cache keyword existence checks
   - Reduces database reads for frequently accessed data
   - AWS Free Tier: 750 hours of t2.micro Redis node per month

2. **Asynchronous Processing**:
   - Moves heavy computation to background workers via SQS
   - Prevents API timeouts for users with many job listings
   - AWS Free Tier: 1 million SQS requests per month

3. **Batch Processing & Pagination**:
   - Retrieves job listings in smaller batches (e.g., 25 at a time)
   - Stays under DynamoDB read capacity unit limits
   - AWS Free Tier: 25 read/write capacity units for DynamoDB

4. **Progressive Loading Pattern**:
   - Returns immediate response to user without waiting for counts
   - Uses WebSockets or Server-Sent Events for real-time updates
   - Improves perceived performance while reducing costs

5. **Database Optimization**:
   - Uses atomic counter updates instead of full record updates
   - Minimizes write capacity units consumed
   - Reduces the risk of write conflicts

## Resilience Improvements

The detailed implementation also includes several resilience enhancements:

1. **Fault Isolation**:
   - Separates keyword creation from count calculation
   - Ensures basic functionality works even if counting fails

2. **Retry Mechanisms**:
   - Queue-based processing enables automatic retries for failed operations
   - Can configure Dead Letter Queues for operations that fail repeatedly

3. **Cache Fallbacks**:
   - System degrades gracefully by falling back to database if cache fails
   - Cache misses automatically populate the cache for future requests

4. **Real-time Updates**:
   - WebSocket/SSE connection provides real-time feedback
   - Users don't need to refresh to see updated counts 