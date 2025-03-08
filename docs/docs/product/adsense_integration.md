# AdSense Integration

This document details the implementation of Google AdSense in Tributary and revenue projections.

## Conservative AdSense Revenue Projections

**For Initial MVP Phase (First 6 Months):**

| Metric | Conservative Estimate | Explanation |
|--------|----------------------|-------------|
| Active Users | 100-300 | Limited initial adoption |
| Monthly Page Views/User | 10 | Basic engagement |
| Ad Impressions/Page | 2 | Top & bottom placement |
| Total Monthly Impressions | 2,000-6,000 | Users × Views × Ad Placements |
| Initial CPM (£) | £0.80 | Lower early CPM for new sites |
| Monthly Ad Revenue | £1.60-£4.80 | Impressions × CPM ÷ 1000 |

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

## Strategic Ad Placement 

For the MVP, we focus on just two strategic ad placements to maximize revenue with minimal user experience impact:

1. **Top Banner (Below Header)**
   - High visibility position
   - First thing users see after the header
   - Size: Responsive leaderboard (728×90 on desktop, adaptive on mobile)
   - Estimated CTR: 0.5-1.0%

2. **Content End (Bottom of Main Content)**
   - Placed after users engage with content
   - Less disruptive to the user experience
   - Size: Medium rectangle (300×250)
   - Estimated CTR: 0.3-0.8%

## Ad Optimization Techniques

To improve AdSense performance without added development costs:

1. **Placement Optimization**
   - Position ads near relevant content 
   - Ensure ads are visible without scrolling
   - Use contrasting colors for ad containers

2. **Ad Format Selection**
   - Use responsive ad units that adapt to screen size
   - Enable both text and display ads
   - Allow auto-sizing to maximize fill rate

3. **Page Content Enhancement**
   - Include job-related keywords in page content
   - Create content sections with industry terminology
   - Use proper semantic HTML to help Google understand context

4. **Compliance Best Practices**
   - Clearly distinguish ads from content
   - Never encourage accidental clicks
   - Maintain adequate spacing between ad units
   - Follow all AdSense policies to avoid account issues

## AdSense Account Setup

### Registration Process

1. Create Google AdSense account at [google.com/adsense](https://www.google.com/adsense)
2. Enter website details and contact information
3. Place AdSense code in site `<head>` for verification
4. Wait for account approval (typically 1-3 days)

### Initial Configuration

1. Create responsive ad units for each position
2. Set up appropriate ad sizes and formats
3. Configure ad category blocking for irrelevant categories
4. Establish URL channel for each section of the site

## Performance Monitoring

Track key metrics to optimize ad performance:

1. **Page RPM (Revenue Per Mille)**
   - Target: Increase from initial £0.80 to £2.00+ over 6 months
   - Action: A/B test ad placements and formats

2. **Click-Through Rate (CTR)**
   - Target: 0.5%+ average across all placements
   - Action: Test different ad formats and positions

3. **Fill Rate**
   - Target: 90%+ ad fill rate
   - Action: Enable all ad formats and sizes

## Zero-Cost Growth Strategies

Methods to increase ad revenue without additional spending:

1. **SEO Optimization**
   - Optimize for job-related keywords
   - Create indexable content pages
   - Improve site performance metrics

2. **Content Strategy**
   - Add job market insights articles
   - Create "how-to" guides for job seekers
   - Publish keyword trend reports

3. **Community Building**
   - Encourage social sharing of insights
   - Create referral mechanisms
   - Build an email list for regular engagement 