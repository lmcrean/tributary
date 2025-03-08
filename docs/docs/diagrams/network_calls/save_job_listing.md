# Save Job Listing

This document outlines the network calls and data flow for saving a job listing in the Tributary application.

## Overview Diagram

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryTextColor': '#ffffff', 'primaryColor': '#ffffff', 'noteBkgColor': '#2b2b2b', 'noteTextColor': '#ffffff', 'activationBorderColor': '#ffffff', 'signalColor': '#ffffff' }}}%%
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Database

    User->>Frontend: Pastes job ad text (Ctrl+V)
    Frontend->>Frontend: Displays job in editable text box
    Note over Frontend: First line becomes custom heading
    User->>Frontend: Clicks "Save Job" button
    Frontend->>API: POST /api/jobs/save
    API->>Database: Get user's tracked keywords
    Database-->>API: Return user's keyword list
    API->>API: Search job text for user's tracked keywords only
    API->>Database: Save job listing
    API->>Database: Update keyword counts (batch operation)
    Database-->>API: Confirm save
    API-->>Frontend: Return success + updated keyword stats
    Frontend-->>User: Show "Job saved" confirmation
    Frontend->>Frontend: Update keyword counts in UI
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

    User->>Frontend: Pastes job ad text (Ctrl+V)
    Frontend->>Frontend: Displays job in editable text box
    Note over Frontend: First line becomes custom heading
    User->>Frontend: Clicks "Save Job" button
    Frontend->>API: POST /api/jobs/save
    
    API->>Cache: Check for user's tracked keywords
    Cache-->>API: Return cache hit/miss
    
    alt Cache Miss
        API->>Database: Get user's tracked keywords
        Database-->>API: Return user's keyword list
        API->>Cache: Update cache with keywords
    end
    
    API->>Database: Save job listing
    Database-->>API: Confirm save with listingId
    
    API->>Queue: Enqueue keyword processing task
    Note over Queue: Decouples job saving from keyword processing
    API-->>Frontend: Return success (initial response)
    Frontend-->>User: Show "Job saved" confirmation
    
    Queue->>Worker: Process keyword search task
    
    Worker->>Cache: Check for user's tracked keywords
    Cache-->>Worker: Return cache hit/miss
    
    alt Cache Miss
        Worker->>Database: Get user's tracked keywords
        Database-->>Worker: Return user's keyword list
        Worker->>Cache: Update cache with keywords
    end
    
    Worker->>Database: Get job listing text
    Database-->>Worker: Return job listing
    
    Worker->>Worker: Initialize keyword counts
    
    loop Until all keywords processed (batch size: 25)
        Worker->>Worker: Process next batch of keywords
        Worker->>Worker: Count keyword mentions in job text
        Note over Worker: Stays within AWS free tier limits
    end
    
    Worker->>Database: Update keyword counts (atomic batch operation)
    Database-->>Worker: Confirm update
    
    Worker->>Frontend: Push updated keyword stats via WebSocket/SSE
    Frontend->>Frontend: Update keyword counts in UI
```

## Data Payload Example

```json
// POST /api/jobs/save request
{
  "heading": "Senior Developer at Acme Inc",
  "fullText": "At Acme Inc, we are looking for a Senior Developer with 5 years of experience in..."
}

// Initial success response (immediate)
{
  "success": true,
  "listingId": 456,
  "message": "Job listing saved, processing keywords..."
}

// WebSocket/SSE update (async)
{
  "type": "KEYWORD_COUNT_UPDATED",
  "listingId": 456,
  "keywordStats": [
    { "keyword": "JavaScript", "newCount": 12 },
    { "keyword": "React", "newCount": 8 },
    { "keyword": "Python", "newCount": 5 },
    { "keyword": "SQL", "newCount": 3 },
    { "keyword": "AWS", "newCount": 2 }
  ]
}
```

## AWS Free Tier & Cost Optimizations

The detailed implementation diagram incorporates several optimizations to ensure the application stays within AWS free tier limits and minimizes costs:

1. **Caching Layer**:
   - Uses ElastiCache (Redis) to cache user's keyword lists
   - Reduces repeated database reads for the same user
   - AWS Free Tier: 750 hours of t2.micro Redis node per month

2. **Asynchronous Processing**:
   - Moves keyword processing to background workers via SQS
   - Prevents API timeouts for large job listings or many keywords
   - AWS Free Tier: 1 million SQS requests per month

3. **Progressive Response Pattern**:
   - Returns immediate confirmation of job save without waiting for keyword processing
   - Uses WebSockets or Server-Sent Events for pushing keyword stats updates
   - Improves perceived performance and user experience

4. **Database Optimization**:
   - Uses atomic batch updates for keyword counts
   - Minimizes the number of write operations to the database
   - Stays within DynamoDB write capacity unit limits (AWS Free Tier: 25 WCUs)

## Resilience Improvements

The detailed implementation also includes several resilience enhancements:

1. **Fault Isolation**:
   - Separates job saving from keyword processing
   - Ensures jobs are saved even if keyword processing fails

2. **Retry Mechanisms**:
   - Queue-based processing enables automatic retries for failed operations
   - Can configure Dead Letter Queues for operations that fail repeatedly

3. **Cache Fallbacks**:
   - System degrades gracefully by falling back to database if cache fails
   - Cache misses automatically populate the cache for future requests

4. **Real-time Updates**:
   - WebSocket/SSE connection provides real-time feedback
   - Updates UI as soon as keyword processing completes 