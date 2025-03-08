# Network Calls and Data Flow

This document outlines the network calls and data flow for key user actions in the Tributary application. The diagrams focus on optimizing for cost efficiency while maintaining necessary functionality.

## User Saves a Job Listing

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

### Data Payload Example

```json
// POST /api/jobs/save request
{
  "heading": "Senior Developer at Acme Inc",
  "fullText": "At Acme Inc, we are looking for a Senior Developer with 5 years of experience in...",
}

// Success response
{
  "success": true,
  "listingId": 456,
  "keywordStats": [
    { "keyword": "JavaScript", "newCount": 12, "status": "have" },
    { "keyword": "React", "newCount": 8, "status": "learning" },
    { "keyword": "Python", "newCount": 5, "status": "need" },
    { "keyword": "SQL", "newCount": 3, "status": "have" },
    { "keyword": "AWS", "newCount": 2, "status": "learning" }
  ]
}
```

### User Creates a Keyword

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryTextColor': '#ffffff', 'primaryColor': '#ffffff', 'noteBkgColor': '#2b2b2b', 'noteTextColor': '#ffffff', 'activationBorderColor': '#ffffff', 'signalColor': '#ffffff' }}}%%
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Database

    User->>Frontend: Clicks "+ Add Custom Keyword" button
    Frontend->>Frontend: Displays keyword creation modal
    User->>Frontend: Enters keyword name, selects status, color
    User->>Frontend: Clicks "Save Keyword" button
    Frontend->>API: POST /api/keywords/create
    API->>Database: Check if keyword already exists for user
    Database-->>API: Return existence check result
    
    alt Keyword already exists
        API-->>Frontend: Return error (duplicate keyword)
        Frontend-->>User: Show error message
    else Keyword doesn't exist
        API->>Database: Save new keyword
        API->>Database: Get user's job listings
        Database-->>API: Return job listings
        API->>API: Calculate keyword mentions across job listings
        API->>Database: Update keyword totalMentions
        Database-->>API: Confirm update
        API-->>Frontend: Return success + keyword data
        Frontend-->>User: Show "Keyword created" confirmation
        Frontend->>Frontend: Add keyword to tracker UI
    end
```

### Data Payload Example

```json
// POST /api/keywords/create request
{
  "name": "TypeScript",
  "status": "learning",
  "colorCode": "#FFCC00"
}

// Success response
{
  "success": true,
  "keyword": {
    "keywordId": 123,
    "name": "TypeScript",
    "status": "learning",
    "colorCode": "#FFCC00",
    "isCustom": true,
    "totalMentions": 3
  }
}
```

### Cost Optimization Strategies

1. **Selective Keyword Processing**:
   - Only extract and process keywords the user is already tracking
   - No processing of new potential keywords until user explicitly adds them

2. **Batch Database Operations**:
   - Update keyword counts in a single database transaction
   - Avoid multiple separate database calls

3. **Minimal Data Transfer**:
   - Send only essential job data in the initial save request
   - Return only updated keyword stats, not the entire job listing

4. **Caching**:
   - Cache frequently accessed user keyword lists
   - Cache job listing check results to prevent redundant database queries

5. **Lazy Loading**:
   - Only calculate totalMentions when specifically requested
   - Defer heavy computations until needed
