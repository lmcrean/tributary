# Tier Comparison

This document details the feature differences between Tributary's free and premium tiers.

## Feature Comparison Table

| Feature | Free Tier | Premium Tier (£1.99/month) |
|---------|-----------|----------------------------|
| **Saved Job Listings** | 5 max | Unlimited |
| **Tracked Keywords** | 10 max | Unlimited |
| **Keyword Analytics** | Basic counts only | Advanced trends + Popular skill analysis |
| **API Requests** | 50/day | 500/day |
| **Ads** | Yes (Google AdSense) | No ads |
| **Support** | Community only | Priority email support |
| **Data Retention** | 1 month | Unlimited |
| **Custom Categories** | No | Yes |
| **Cross-device Sync** | Limited | Full sync across all devices |

## Free Tier Technical Implementation

```javascript
// Reference implementation for free tier limits
const FREE_TIER_LIMITS = {
  SAVED_JOBS_MAX: 5,
  TRACKED_KEYWORDS_MAX: 10,
  API_REQUESTS_PER_DAY: 50,
  DATA_RETENTION_DAYS: 30,
  MAX_JOB_TEXT_LENGTH: 10000, // Characters
  MAX_STORAGE_PER_USER_KB: 100,
  AVAILABLE_FEATURES: [
    'basic_keyword_counts',
    'job_saving',
    'keyword_tracking_basic',
    'mobile_view'
  ]
};

// Example middleware for limit enforcement
function checkFreeTierLimits(req, res, next) {
  const userId = req.user.id;
  const userTier = req.user.tier;
  
  if (userTier === 'premium') {
    return next(); // No limits for premium users
  }
  
  // Check relevant limit based on request type
  // Implementation details would vary based on specific endpoint
  // ...
}
```

## Premium Features Explained

### 1. Unlimited Job Listings & Keywords

- No artificial caps on the number of jobs or keywords
- Ensures power users can track their entire job search

### 2. Advanced Analytics

- Trend analysis showing keyword popularity over time
- Industry comparisons of skill requirements
- Personalized skill gap identification

### 3. Enhanced Job Analysis

- AI-powered job requirement extraction
- Salary range estimation
- Level/seniority classification
- Automatic company research

### 4. Export Functionality

- Export job listings to CSV or PDF
- Create custom reports for job search tracking
- Export keyword statistics for portfolio building

## Premium Tier Conversion Strategy

Once the MVP has validated user interest and the product concept, premium conversion becomes viable:

### 1. Soft Prompts

- Usage meters showing approach to free limits
- Feature previews with upgrade buttons

### 2. Hard Limits

- Clear messaging when limits are reached
- One-click upgrade option

### 3. Value Proposition

- Focus on career advancement value
- Emphasize time savings and organization
- Highlight premium-only insights 