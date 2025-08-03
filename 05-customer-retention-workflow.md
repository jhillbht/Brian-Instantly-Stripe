# Customer Retention Workflow - Stripe to Instantly Integration

## Business Overview

This integration creates an automated customer lifecycle marketing system that converts Stripe customers into Instantly leads for annual renewal reminders and cross-selling opportunities. Perfect for businesses with annual services like LLC registrations, business licenses, or recurring professional services.

## Workflow Specification

### Trigger Event
**Stripe Webhook**: `checkout.session.completed` or `payment_intent.succeeded`

### Data Extraction Requirements
From each Stripe purchase, extract:
- ✅ **Customer Email** (`customer.email`)
- ✅ **First Name** (`customer.name` - parsed first part)
- ✅ **Last Name** (`customer.name` - parsed last part)
- ✅ **Product Purchased** (`line_items[0].description` or `product.name`)
- ✅ **Purchase Date** (`created` timestamp)
- ✅ **Billing State** (`customer.address.state` or `billing_address.state`)

### Campaign Assignment Logic
**Campaign Name Format**: `[State] [Month] [Day] [Year]`

**Examples**:
- Purchase on April 1st, 2024 in Virginia → **"Virginia April 1st 2025"**
- Purchase on December 15th, 2024 in California → **"California December 15th 2025"**
- Purchase on January 3rd, 2024 in Texas → **"Texas January 3rd 2025"**

### Business Logic Rules

#### Campaign Creation
1. **Auto-Create Campaigns**: If campaign doesn't exist, create it automatically
2. **Campaign Naming**: Always use the format `[State] [Month] [Day] [Year]`
3. **Year Logic**: Use the year FOLLOWING the purchase year for renewal reminders

#### Lead Management
1. **Duplicate Handling**: Check if customer email already exists in the campaign
2. **Lead Updates**: If exists, update purchase history; if new, create lead
3. **Custom Fields**: Store product purchased, original purchase date, purchase amount

#### Segmentation Strategy
- **By State**: Each state gets separate campaigns for compliance/legal reasons
- **By Date**: Specific purchase dates for precise annual timing
- **By Product**: Custom sequences based on what was purchased

## Data Flow Architecture

```
Stripe Purchase
      ↓
Webhook Triggered
      ↓
Extract Customer Data
      ↓
Parse State & Date
      ↓
Generate Campaign Name
      ↓
Check/Create Campaign in Instantly
      ↓
Add Customer as Lead
      ↓
Trigger Welcome Sequence
      ↓
Schedule Annual Reminder (11 months)
```

## Implementation Requirements

### Webhook Endpoint Setup
```javascript
// Webhook handler for Stripe events
app.post('/stripe-webhook', (req, res) => {
  const event = req.body;
  
  if (event.type === 'checkout.session.completed' || 
      event.type === 'payment_intent.succeeded') {
    
    processCustomerPurchase(event.data.object);
  }
  
  res.status(200).send('OK');
});
```

### Data Processing Function
```javascript
async function processCustomerPurchase(stripeData) {
  // Extract customer information
  const customerData = {
    email: stripeData.customer_email || stripeData.customer.email,
    fullName: stripeData.customer.name,
    firstName: parseFirstName(stripeData.customer.name),
    lastName: parseLastName(stripeData.customer.name),
    product: stripeData.line_items[0].description,
    purchaseDate: new Date(stripeData.created * 1000),
    billingState: stripeData.customer.address?.state,
    purchaseAmount: stripeData.amount_total / 100
  };
  
  // Generate campaign name
  const campaignName = generateCampaignName(
    customerData.billingState, 
    customerData.purchaseDate
  );
  
  // Add to Instantly
  await addToInstantlyCampaign(customerData, campaignName);
}
```

### Campaign Name Generation
```javascript
function generateCampaignName(state, purchaseDate) {
  const nextYear = purchaseDate.getFullYear() + 1;
  const month = purchaseDate.toLocaleString('default', { month: 'long' });
  const day = purchaseDate.getDate();
  
  // Add ordinal suffix (1st, 2nd, 3rd, etc.)
  const dayWithSuffix = addOrdinalSuffix(day);
  
  return `${state} ${month} ${dayWithSuffix} ${nextYear}`;
}

function addOrdinalSuffix(day) {
  if (day >= 11 && day <= 13) return `${day}th`;
  
  switch (day % 10) {
    case 1: return `${day}st`;
    case 2: return `${day}nd`;
    case 3: return `${day}rd`;
    default: return `${day}th`;
  }
}
```

## Use Cases & Benefits

### Primary Use Case: Annual Renewal Reminders
- **LLC Registration Services**: Remind customers to renew their LLC annually
- **Business License Renewals**: State-specific renewal requirements
- **Professional Service Renewals**: CPA, legal, consulting services

### Secondary Use Case: Cross-Selling Opportunities
- **Related Services**: Offer complementary business services
- **Upgrade Opportunities**: Premium packages or additional features
- **Referral Programs**: Encourage customer referrals

### Benefits for Warm Audience Marketing
- ✅ **High Engagement**: Past customers more likely to open emails
- ✅ **Better Deliverability**: Not cold outreach, reduces spam risk
- ✅ **Targeted Messaging**: State-specific and product-specific content
- ✅ **Automated Timing**: Perfect annual reminder timing
- ✅ **Revenue Recovery**: Capture renewals that might be forgotten

## Success Metrics

### Email Performance (Expected Higher Due to Warm Audience)
- **Open Rates**: Target 45-65% (higher than cold outreach)
- **Click Rates**: Target 8-15% (engaged past customers)
- **Reply Rates**: Target 5-12% (familiar with your business)

### Business Metrics
- **Renewal Rate**: Track annual renewal percentage
- **Cross-sell Conversion**: Additional services sold to existing customers
- **Customer Lifetime Value**: Revenue per customer over time
- **Revenue Attribution**: Income generated from email campaigns

### Operational Metrics
- **Campaign Creation Speed**: Auto-creation of new campaigns
- **Data Sync Accuracy**: Stripe to Instantly data mapping success
- **Timing Precision**: Annual reminders sent at optimal times

## Risk Mitigation

### Data Privacy Compliance
- **Customer Consent**: Ensure purchase includes email marketing consent
- **Unsubscribe Options**: Easy opt-out from renewal reminders
- **Data Retention**: Comply with state and federal data retention laws

### Email Deliverability
- **Sender Reputation**: Monitor email sending reputation
- **Content Quality**: Professional, helpful reminder content
- **Frequency Management**: Avoid over-emailing past customers

### Business Continuity
- **Backup Systems**: Alternative reminder methods if email fails
- **Manual Override**: Ability to manually manage critical renewals
- **Error Handling**: Graceful failure handling for missed webhooks

## Next Steps for Implementation

1. **Set Up Stripe Webhooks** for purchase events
2. **Configure Instantly API** for campaign and lead management
3. **Build Data Processing Logic** for customer extraction and formatting
4. **Create Campaign Templates** for different states and products
5. **Design Email Sequences** for welcome and annual reminder flows
6. **Test with Sample Data** before processing live customers
7. **Monitor and Optimize** campaign performance over time

This system transforms one-time customers into a valuable, recurring revenue stream through automated lifecycle marketing.