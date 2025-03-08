# Subscription Management

This document outlines the network calls and data flow for managing subscriptions in the Tributary application.

## Overview Diagram

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryTextColor': '#ffffff', 'primaryColor': '#ffffff', 'noteBkgColor': '#2b2b2b', 'noteTextColor': '#ffffff', 'activationBorderColor': '#ffffff', 'signalColor': '#ffffff' }}}%%
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Stripe
    participant Database

    %% Subscribe Flow
    User->>Frontend: Clicks "Subscribe" button
    Frontend->>API: POST /api/subscription/create-checkout
    API->>Stripe: Create checkout session
    Stripe-->>API: Return checkout session ID
    API-->>Frontend: Return checkout session ID
    Frontend->>Stripe: Redirect to Stripe checkout page
    User->>Stripe: Completes payment
    Stripe->>API: Webhook: payment_intent.succeeded
    API->>Database: Create/Update subscription
    Database-->>API: Confirm subscription saved
    Stripe->>Frontend: Redirect to success page
    Frontend-->>User: Show subscription confirmation

    %% Cancel Flow
    User->>Frontend: Clicks "Cancel Subscription"
    Frontend->>API: POST /api/subscription/cancel
    API->>Stripe: Cancel subscription
    Stripe-->>API: Confirm cancellation
    API->>Database: Update subscription status
    Database-->>API: Confirm update
    API-->>Frontend: Return success
    Frontend-->>User: Show cancellation confirmation
```

## Detailed Implementation Diagram

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryTextColor': '#ffffff', 'primaryColor': '#ffffff', 'noteBkgColor': '#2b2b2b', 'noteTextColor': '#ffffff', 'activationBorderColor': '#ffffff', 'signalColor': '#ffffff' }}}%%
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Cache
    participant Stripe
    participant Queue
    participant Worker
    participant Database

    %% Subscribe Flow
    User->>Frontend: Clicks "Subscribe" button
    Frontend->>API: POST /api/subscription/create-checkout
    API->>API: Validate user authentication
    API->>Stripe: Create checkout session
    Note over API,Stripe: Include success/cancel URLs
    Stripe-->>API: Return checkout session ID
    API-->>Frontend: Return checkout session ID
    Frontend->>Stripe: Redirect to Stripe checkout page
    User->>Stripe: Completes payment information
    Stripe-->>User: Processes payment
    
    alt Payment Successful
        Stripe->>API: Webhook: payment_intent.succeeded
        API->>Queue: Enqueue subscription processing
        Queue->>Worker: Process subscription creation
        Worker->>Database: Create/Update subscription record
        Worker->>Cache: Invalidate user subscription cache
        Database-->>Worker: Confirm subscription saved
        Stripe->>Frontend: Redirect to success URL
        Frontend->>API: GET /api/subscription/status
        API->>Cache: Check subscription status
        Cache-->>API: Return cache hit/miss
        
        alt Cache Miss
            API->>Database: Get subscription status
            Database-->>API: Return subscription data
            API->>Cache: Update subscription cache
        end
        
        API-->>Frontend: Return subscription details
        Frontend-->>User: Show subscription confirmation
    else Payment Failed
        Stripe->>Frontend: Redirect to cancel URL
        Frontend-->>User: Show payment failure message
    end

    %% Cancel Flow
    User->>Frontend: Clicks "Cancel Subscription"
    Frontend->>Frontend: Show confirmation dialog
    User->>Frontend: Confirms cancellation
    Frontend->>API: POST /api/subscription/cancel
    API->>API: Validate user authentication
    API->>Stripe: Cancel subscription (at period end)
    Note over API,Stripe: Allows continued access until billing period ends
    Stripe-->>API: Confirm cancellation
    API->>Queue: Enqueue cancellation processing
    Queue->>Worker: Process subscription cancellation
    Worker->>Database: Update subscription status to "canceled"
    Worker->>Database: Set cancelDate field
    Worker->>Cache: Invalidate user subscription cache
    Database-->>Worker: Confirm update
    API-->>Frontend: Return initial success
    Frontend-->>User: Show initial cancellation confirmation
    
    %% Subscription Status Check
    User->>Frontend: Visits account page
    Frontend->>API: GET /api/subscription/status
    API->>Cache: Check subscription status
    Cache-->>API: Return cache hit/miss
    
    alt Cache Miss
        API->>Database: Get subscription status
        Database-->>API: Return subscription data
        API->>Cache: Update subscription cache
    end
    
    API-->>Frontend: Return detailed subscription info
    Frontend-->>User: Display subscription details
```

## Data Payload Examples

```json
// POST /api/subscription/create-checkout request
{
  "priceId": "price_1AbCdEfGhIjKlMnOpQrStUvW",
  "successUrl": "https://tributary.app/subscription/success",
  "cancelUrl": "https://tributary.app/subscription/cancel"
}

// POST /api/subscription/create-checkout response
{
  "success": true,
  "checkoutSessionId": "cs_test_a1b2c3d4e5f6g7h8i9j0",
  "checkoutUrl": "https://checkout.stripe.com/c/pay/cs_test_a1b2c3d4e5f6g7h8i9j0"
}

// Stripe webhook event: payment_intent.succeeded
{
  "id": "evt_1AbCdEfGhIjKlMnOpQrStUvW",
  "object": "event",
  "type": "payment_intent.succeeded",
  "data": {
    "object": {
      "id": "pi_1AbCdEfGhIjKlMnOpQrStUvW",
      "customer": "cus_1AbCdEfGhIjKlMnOpQrSt",
      "amount": 199,
      "currency": "gbp",
      "status": "succeeded"
    }
  }
}

// POST /api/subscription/cancel request
{
  "cancelImmediately": false
}

// POST /api/subscription/cancel response
{
  "success": true,
  "message": "Subscription will be canceled at the end of the current billing period",
  "effectiveDate": "2023-06-15T00:00:00Z"
}

// GET /api/subscription/status response
{
  "success": true,
  "subscription": {
    "status": "active", // or "canceled", "past_due", etc.
    "currentPeriodStart": "2023-05-15T00:00:00Z",
    "currentPeriodEnd": "2023-06-15T00:00:00Z",
    "cancelAtPeriodEnd": false,
    "plan": {
      "name": "Premium",
      "amount": 199,
      "currency": "gbp",
      "interval": "month"
    }
  }
}
```

## AWS Free Tier & Cost Optimizations

The subscription management implementation incorporates several optimizations to ensure the application stays within AWS free tier limits:

1. **Caching Layer**:
   - Uses ElastiCache (Redis) to cache subscription status
   - Reduces repeated database reads
   - AWS Free Tier: 750 hours of t2.micro Redis node per month

2. **Asynchronous Processing**:
   - Processes subscription events via background workers
   - Ensures system remains responsive
   - AWS Free Tier: 1 million SQS requests per month

3. **Webhook Management**:
   - Uses API Gateway for Stripe webhook endpoints
   - AWS Free Tier: 1 million API calls per month

4. **Database Optimization**:
   - Minimal subscription data stored in database
   - Leverages Stripe for payment history details
   - Stays within DynamoDB read/write capacity unit limits

## Resilience Improvements

The implementation includes several resilience features:

1. **Idempotent Processing**:
   - Webhook handlers are idempotent to prevent duplicate processing
   - Using Stripe's idempotency keys for payment operations

2. **Graceful Degradation**:
   - System falls back to database if cache is unavailable
   - Frontend displays appropriate messages during service disruptions

3. **Retry Mechanisms**:
   - Queue-based processing enables automatic retries for failed operations
   - Configured Dead Letter Queues for handling persistent failures

4. **Error Recovery**:
   - Subscription state can be manually reconciled through admin tools
   - Regular audits compare Stripe subscription state with database
``` 