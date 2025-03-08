# AWS Implementation

This document details the AWS architecture and cost control mechanisms for Tributary's MVP.

## Serverless-First Architecture

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

## AWS Free Tier Maximization

For the MVP, we'll aggressively optimize for AWS Free Tier offerings:

| AWS Service | Free Tier Allowance | Our Enforced Limit | Cost Control Method |
|-------------|---------------------|-------------------|----------------------|
| Lambda | 1M requests/month | 800K requests/month | API Gateway throttling |
| DynamoDB | 25 WCU, 25 RCU | 20 WCU, 20 RCU | Small item size, compression |
| S3 | 5GB storage | 1GB storage | Enforced user storage limits |
| API Gateway | 1M requests/month | 800K requests/month | Per-user throttling |
| CloudFront | 1TB data transfer | 800GB | Cache optimization, compression |
| CloudWatch | 10 metrics, 10 alarms | 8 metrics, 8 alarms | Focus on critical metrics only |

## Automated Cost Control System

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

## Emergency Shutdown Procedure

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

## AWS Cost Optimization Strategies

### 1. Data Storage Optimization

- Compress job listing text (gzip)
- Store only essential fields
- Implement automatic cleanup of old data
- Enforce character limits on all text fields

### 2. API Request Reduction

- Implement aggressive client-side caching
- Batch operations whenever possible
- Reduce polling frequency for updates

### 3. Lambda Optimization

- Minimum memory allocation (128MB where possible)
- Short timeouts (3s maximum)
- Cold start optimization
- Share connections across invocations

### 4. CloudFront Optimization

- Long cache TTLs for static content
- Serve compressed content only

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