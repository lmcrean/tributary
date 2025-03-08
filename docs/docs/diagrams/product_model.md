# Tributary Product Model

This document outlines Tributary's monetization strategy, tier limits, and operational cost coverage.

## Two-Tier Freemium Model

Tributary uses a freemium model with two tiers:

1. **Free Tier with Ads**: Basic functionality with strict limits and Google AdSense integration
2. **Premium Tier (£1.99/month)**: Expanded functionality, higher limits, and no ads

## Tier Comparison

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

## Free Tier Limitations (Technical Implementation)

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

1. **Unlimited Job Listings & Keywords**
   - No artificial caps on the number of jobs or keywords
   - Ensures power users can track their entire job search

2. **Advanced Analytics**
   - Trend analysis showing keyword popularity over time
   - Industry comparisons of skill requirements
   - Personalized skill gap identification

3. **Enhanced Job Analysis**
   - AI-powered job requirement extraction
   - Salary range estimation
   - Level/seniority classification
   - Automatic company research

4. **Export Functionality**
   - Export job listings to CSV or PDF
   - Create custom reports for job search tracking
   - Export keyword statistics for portfolio building

## MVP Cost Structure & Strict £50 Budget Cap

### Serverless-First Architecture

To minimize costs, we'll use a serverless-first approach with AWS Free Tier services only:

| AWS Service | Implementation Approach | Cost Control |
|-------------|-------------------------|--------------|
| **Lambda** | Primary compute method | Hard limits at 80% of free tier |
| **DynamoDB** | On-demand capacity only | Strict item size limits |
| **S3** | Static website hosting | Max 1GB storage enforcement |
| **API Gateway** | API endpoints | Request throttling at free tier limits |
| **CloudFront** | Content delivery | Restrict to required regions only |
| **CloudWatch** | Cost alarms | Set at £10, £25, £40 thresholds |
| **Cognito** | User authentication | Stay within free tier (50K MAU) |

**Avoided Services:** EC2, RDS, Elastic Beanstalk, or any service with hourly charges

### AWS Free Tier Maximization

For the MVP, we'll aggressively optimize for AWS Free Tier offerings:

| AWS Service | Free Tier Allowance | Our Enforced Limit | Cost Control Method |
|-------------|---------------------|-------------------|----------------------|
| Lambda | 1M requests/month | 800K requests/month | API Gateway throttling |
| DynamoDB | 25 WCU, 25 RCU | 20 WCU, 20 RCU | Small item size, compression |
| S3 | 5GB storage | 1GB storage | Enforced user storage limits |
| API Gateway | 1M requests/month | 800K requests/month | Per-user throttling |
| CloudFront | 1TB data transfer | 800GB | Cache optimization, compression |
| CloudWatch | 10 metrics, 10 alarms | 8 metrics, 8 alarms | Focus on critical metrics only |

### Strict Budget Timeline

| Month | Target Activities | Budget Allocation | Cumulative Total |
|-------|------------------|-------------------|------------------|
| 1 | AWS setup, initial deployment | £10 | £10 |
| 2 | Core functionality implementation | £10 | £20 |
| 3 | AdSense integration | £0 | £20 |
| 4 | Limited promotion, bug fixes | £10 | £30 |
| 5 | Additional features if budget allows | £10 | £40 |
| 6 | Reserve for unexpected costs | £10 | £50 |

### Automated Cost Control System

```javascript
// AWS Lambda function for automated cost control (runs daily)
exports.costGuardian = async (event) => {
  const AWS = require('aws-sdk');
  const cloudwatch = new AWS.CloudWatch();
  
  // Get current month AWS cost
  const costExplorer = new AWS.CostExplorer();
  const today = new Date();
  const startDate = new Date(today.getFullYear(), today.getMonth(), 1);
  
  const costData = await costExplorer.getCostAndUsage({
    TimePeriod: {
      Start: startDate.toISOString().split('T')[0],
      End: today.toISOString().split('T')[0]
    },
    Granularity: 'MONTHLY',
    Metrics: ['UnblendedCost']
  }).promise();
  
  const currentCost = parseFloat(costData.ResultsByTime[0].Total.UnblendedCost.Amount);
  
  // Implement tiered restrictions based on current spend
  if (currentCost > 40) {  // £40+ - Critical mode
    await setServiceLimits('CRITICAL');
    await notifyAdmin('AWS costs have exceeded £40! Extreme restrictions applied.');
    return { status: 'CRITICAL_RESTRICTIONS_APPLIED' };
  } 
  else if (currentCost > 25) {  // £25-40 - Severe restrictions
    await setServiceLimits('SEVERE');
    await notifyAdmin('AWS costs have exceeded £25. Severe restrictions applied.');
    return { status: 'SEVERE_RESTRICTIONS_APPLIED' };
  }
  else if (currentCost > 10) {  // £10-25 - Moderate restrictions
    await setServiceLimits('MODERATE');
    return { status: 'MODERATE_RESTRICTIONS_APPLIED' };
  }
  
  return { status: 'NORMAL_OPERATIONS', currentCost };
};

// Set resource limits based on cost tier
async function setServiceLimits(tier) {
  // Implementation to adjust service limits based on spend tiers
  const limitConfig = {
    NORMAL: { jobsPerUser: 5, keywordsPerUser: 10, apiCallsPerDay: 50 },
    MODERATE: { jobsPerUser: 3, keywordsPerUser: 7, apiCallsPerDay: 30 },
    SEVERE: { jobsPerUser: 2, keywordsPerUser: 5, apiCallsPerDay: 20 },
    CRITICAL: { jobsPerUser: 1, keywordsPerUser: 3, apiCallsPerDay: 10 }
  };
  
  // Apply the selected limits
  // ...
}
```

### Emergency Shutdown Procedure

```javascript
// Emergency shutdown Lambda (triggered by CloudWatch alarm at £45)
exports.emergencyShutdown = async (event) => {
  const AWS = require('aws-sdk');
  
  // Disable API Gateway stages
  const apigateway = new AWS.APIGateway();
  await apigateway.updateStage({
    restApiId: process.env.API_ID,
    stageName: 'prod',
    patchOperations: [{
      op: 'replace',
      path: '/*/*/throttling/rateLimit',
      value: '0.1'  // Reduce to minimum rate
    }]
  }).promise();
  
  // Set Lambda concurrency to minimum
  const lambda = new AWS.Lambda();
  const functions = await lambda.listFunctions().promise();
  
  for (const fn of functions.Functions) {
    await lambda.putFunctionConcurrency({
      FunctionName: fn.FunctionName,
      ReservedConcurrentExecutions: 1  // Absolute minimum
    }).promise();
  }
  
  // Send emergency notification
  // ...
  
  return { status: 'EMERGENCY_MEASURES_APPLIED' };
};
```

### Conservative AdSense Revenue Projections

**For Initial MVP Phase (First 6 Months):**

| Metric | Conservative Estimate | Explanation |
|--------|----------------------|-------------|
| Active Users | 100-300 | Limited initial adoption |
| Monthly Page Views/User | 10 | Basic engagement |
| Ad Impressions/Page | 2 | Top & bottom placement |
| Total Monthly Impressions | 2,000-6,000 | Users × Views × Ad Placements |
| Initial CPM (£) | £0.80 | Lower early CPM for new sites |
| Monthly Ad Revenue | £1.60-£4.80 | Impressions × CPM ÷ 1000 |

### AWS Cost Optimization Strategies

1. **Data Storage Optimization**
   - Compress job listing text (gzip)
   - Store only essential fields
   - Implement automatic cleanup of old data
   - Enforce character limits on all text fields

2. **API Request Reduction**
   - Implement aggressive client-side caching
   - Batch operations whenever possible
   - Reduce polling frequency for updates

3. **Lambda Optimization**
   - Minimum memory allocation (128MB where possible)
   - Short timeouts (3s maximum)
   - Cold start optimization
   - Share connections across invocations

4. **CloudFront Optimization**
   - Long cache TTLs for static content
   - Serve compressed content only

## Ad Implementation Specification

```jsx
// Ad component implementation
function AdBanner({ position }) {
  const { userTier } = useUserContext();
  
  // Don't show ads to premium users
  if (userTier === 'premium') return null;
  
  return (
    <div className={`ad-container ad-${position}`} data-testid={`ad-${position}`}>
      <ins className="adsbygoogle"
           style={{ display: 'block' }}
           data-ad-client="ca-pub-XXXXXXXXXXXXXXXX"
           data-ad-slot={getAdSlotForPosition(position)}
           data-ad-format="auto"
           data-full-width-responsive="true"></ins>
    </div>
  );
}

// Ad initialization
useEffect(() => {
  const { userTier } = userContext;
  
  // Only initialize ads for free users
  if (userTier === 'free' && window.adsbygoogle) {
    (window.adsbygoogle = window.adsbygoogle || []).push({});
  }
}, [userContext]);

// Minimal ad placements for MVP (2 strategic locations only)
const AD_POSITIONS = {
  TOP_BANNER: 'top-banner',      // Below header
  CONTENT_END: 'content-end',    // End of main content
};
```

## AWS Cost Optimization Code Examples

```javascript
// Example: DynamoDB Optimization
const AWS = require('aws-sdk');
const dynamoDB = new AWS.DynamoDB.DocumentClient({
  region: 'us-east-1',
  // Use on-demand capacity mode in AWS console
});

// Compress text to reduce storage costs
function compressText(text) {
  const zlib = require('zlib');
  const compressed = zlib.gzipSync(text);
  return compressed.toString('base64');
}

function decompressText(compressed) {
  const zlib = require('zlib');
  const buffer = Buffer.from(compressed, 'base64');
  return zlib.gunzipSync(buffer).toString();
}

// Batch writes to reduce costs
async function batchUpdateKeywords(userId, keywords) {
  // Group updates in batches of 25 (DynamoDB limit)
  const batches = [];
  for (let i = 0; i < keywords.length; i += 25) {
    batches.push(keywords.slice(i, i + 25));
  }
  
  // Process each batch
  for (const batch of batches) {
    const params = {
      RequestItems: {
        'Keywords': batch.map(keyword => ({
          PutRequest: {
            Item: {
              userId,
              keyword: keyword.name,
              // Store minimal data
              count: keyword.count,
              lastUpdated: new Date().toISOString()
            }
          }
        }))
      }
    };
    
    await dynamoDB.batchWrite(params).promise();
  }
}

// Example: Lambda optimization
exports.handler = async (event) => {
  // Use environment variables for config to reduce cold start times
  const { MAX_ITEMS } = process.env;
  
  // Process only what's needed
  const limit = Math.min(event.limit || 10, MAX_ITEMS);
  
  // Return minimal data structure
  return {
    statusCode: 200,
    body: JSON.stringify({ 
      items: results.slice(0, limit),
      // Don't include full result set
    }),
  };
};
```

## Premium Tier Conversion Strategy

Once the MVP has validated user interest and the product concept, premium conversion becomes viable:

1. **Soft Prompts**:
   - Usage meters showing approach to free limits
   - Feature previews with upgrade buttons

2. **Hard Limits**:
   - Clear messaging when limits are reached
   - One-click upgrade option

3. **Value Proposition**:
   - Focus on career advancement value
   - Emphasize time savings and organization
   - Highlight premium-only insights

## Product Validation Decision Points

### Success Metrics (Month 6)

| Metric | Minimum Threshold | Action if Met | Action if Not Met |
|--------|------------------|---------------|-------------------|
| Monthly Active Users | 300+ | Continue development | Evaluate product-market fit |
| Ad Revenue | £5+/month | Scale cautiously | Pivot or sunset |
| User Retention | 30%+ weekly return | Invest in growth | Reassess value proposition |
| Premium Conversions | 5+ users | Scale premium features | Reconsider pricing model |

### Sunset Plan (If Product Fails to Gain Traction)

If the product fails to meet minimum success metrics by month 6:

1. Notify users 30 days in advance of shutdown
2. Provide data export functionality 
3. Decommission AWS resources systematically
4. Document learnings for future projects
