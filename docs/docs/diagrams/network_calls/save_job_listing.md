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
    
    API->>API: Search job text for user's tracked keywords only
    API->>Database: Save job listing
    API->>Database: Update keyword counts (batch operation)
    Database-->>API: Confirm save
    API-->>Frontend: Return success + updated keyword stats
    Frontend-->>User: Show "Job saved" confirmation
    Frontend->>Frontend: Update keyword counts in UI
```

## Data Payload Example

```json
// POST /api/jobs/save request
{
  "heading": "Senior Developer at Acme Inc",
  "fullText": "At Acme Inc, we are looking for a Senior Developer with 5 years of experience in..."
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