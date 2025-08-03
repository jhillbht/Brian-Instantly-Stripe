# Campaign Naming & Organization System

## Overview

The campaign naming system creates organized, predictable campaign structures in Instantly based on customer purchase location and timing. This enables precise annual renewal reminders and state-specific marketing compliance.

## Campaign Naming Convention

### Standard Format
```
[State] [Month] [Day with Suffix] [Year]
```

### Examples by Purchase Date

| Purchase Date | Customer State | Campaign Name |
|---------------|----------------|---------------|
| April 1, 2024 | Virginia | **Virginia April 1st 2025** |
| April 1, 2024 | California | **California April 1st 2025** |
| December 15, 2024 | Texas | **Texas December 15th 2025** |
| January 3, 2024 | Florida | **Florida January 3rd 2025** |
| February 29, 2024 | New York | **New York February 29th 2025** |
| October 31, 2024 | Illinois | **Illinois October 31st 2025** |

### Ordinal Number Rules

| Day | Suffix | Example |
|-----|--------|---------|
| 1 | st | 1st |
| 2 | nd | 2nd |
| 3 | rd | 3rd |
| 4-20 | th | 4th, 15th, 20th |
| 21 | st | 21st |
| 22 | nd | 22nd |
| 23 | rd | 23rd |
| 24-30 | th | 24th, 30th |
| 31 | st | 31st |

**Special Cases (11th, 12th, 13th)**: Always use "th" regardless of the last digit.

## Campaign Organization Strategy

### State-Based Segmentation

#### Why Organize by State?
1. **Legal Compliance**: Different states have different business requirements
2. **Renewal Timing**: State-specific filing deadlines and requirements
3. **Content Customization**: State-specific forms, fees, and procedures
4. **Compliance Tracking**: Monitor state-by-state performance and compliance

#### State Name Handling
- Use full state names, not abbreviations
- **Correct**: "California April 1st 2025"
- **Incorrect**: "CA April 1st 2025"

### Date-Based Precision

#### Why Include Specific Dates?
1. **Renewal Timing**: LLC registrations often have annual requirements
2. **Deadline Awareness**: Customers need reminders before specific dates
3. **Batch Processing**: Group customers with similar renewal timelines
4. **Campaign Performance**: Track effectiveness by renewal timing

#### Date Formatting Rules
- Use full month names: "January", "February", "March", etc.
- Include ordinal suffixes: "1st", "2nd", "3rd", "4th", etc.
- Use 4-digit years: "2025", not "25"

## Campaign Lifecycle Management

### Automatic Campaign Creation

```javascript
async function createCampaignIfNotExists(campaignName) {
  // Check if campaign exists
  const existingCampaign = await instantly.getCampaign(campaignName);
  
  if (!existingCampaign) {
    // Create new campaign
    const newCampaign = await instantly.createCampaign({
      name: campaignName,
      description: `Annual renewal reminders for ${campaignName}`,
      type: 'customer_retention',
      tags: ['renewal', 'annual', extractState(campaignName)],
      status: 'active'
    });
    
    console.log(`Created new campaign: ${campaignName}`);
    return newCampaign;
  }
  
  return existingCampaign;
}
```

### Campaign Templates by Product Type

#### LLC Registration Services
```
Campaign Name: Virginia April 1st 2025
Description: Annual LLC renewal reminders for Virginia customers who registered on April 1st, 2024
Tags: ['llc', 'renewal', 'virginia', 'annual']
Sequence Type: LLC Annual Renewal
```

#### Business License Services
```
Campaign Name: California June 15th 2025
Description: Business license renewal reminders for California customers
Tags: ['license', 'renewal', 'california', 'business']
Sequence Type: License Renewal Reminder
```

#### Professional Services
```
Campaign Name: Texas October 31st 2025
Description: Professional service renewal reminders for Texas customers
Tags: ['professional', 'renewal', 'texas', 'service']
Sequence Type: Service Renewal Follow-up
```

## Data Storage and Tracking

### Campaign Custom Fields

Store additional data for better tracking:

```javascript
const campaignMetadata = {
  state: 'Virginia',
  originalPurchaseDate: '2024-04-01',
  renewalDate: '2025-04-01',
  productType: 'LLC Registration',
  totalCustomers: 0,
  createdDate: new Date().toISOString()
};
```

### Lead Custom Fields

Track individual customer data:

```javascript
const leadCustomFields = {
  originalPurchaseDate: '2024-04-01',
  productPurchased: 'LLC Registration - Virginia',
  purchaseAmount: 297.00,
  renewalDueDate: '2025-04-01',
  customerStatus: 'active',
  lastRenewalReminder: null
};
```

## Campaign Naming Implementation

### JavaScript Implementation

```javascript
class CampaignNamingSystem {
  constructor() {
    this.months = [
      'January', 'February', 'March', 'April', 'May', 'June',
      'July', 'August', 'September', 'October', 'November', 'December'
    ];
  }

  generateCampaignName(state, purchaseDate) {
    const renewalDate = new Date(purchaseDate);
    renewalDate.setFullYear(renewalDate.getFullYear() + 1);
    
    const year = renewalDate.getFullYear();
    const month = this.months[renewalDate.getMonth()];
    const day = this.addOrdinalSuffix(renewalDate.getDate());
    
    return `${state} ${month} ${day} ${year}`;
  }

  addOrdinalSuffix(day) {
    // Special cases for 11th, 12th, 13th
    if (day >= 11 && day <= 13) {
      return `${day}th`;
    }
    
    const lastDigit = day % 10;
    switch (lastDigit) {
      case 1: return `${day}st`;
      case 2: return `${day}nd`;
      case 3: return `${day}rd`;
      default: return `${day}th`;
    }
  }

  extractStateFromCampaign(campaignName) {
    return campaignName.split(' ')[0];
  }

  extractDateFromCampaign(campaignName) {
    const parts = campaignName.split(' ');
    const month = parts[1];
    const dayWithSuffix = parts[2];
    const year = parts[3];
    
    const day = parseInt(dayWithSuffix.replace(/\D/g, ''));
    const monthIndex = this.months.indexOf(month);
    
    return new Date(year, monthIndex, day);
  }
}
```

## Campaign Performance Tracking

### Key Metrics by Campaign

Track performance for each state+date combination:

- **Total Leads**: Number of customers in campaign
- **Email Metrics**: Open rates, click rates, reply rates
- **Renewal Rate**: Percentage who renewed their service
- **Revenue Generated**: Total revenue from renewal reminders
- **Cross-sell Success**: Additional services sold

### Reporting Structure

```javascript
const campaignReport = {
  campaignName: 'Virginia April 1st 2025',
  state: 'Virginia',
  renewalDate: '2025-04-01',
  totalLeads: 247,
  emailsSent: 1235,
  emailsOpened: 617,
  emailsClicked: 123,
  renewalsCompleted: 89,
  renewalRevenue: 26433.00,
  crossSellRevenue: 5940.00,
  campaignROI: 4.2
};
```

## Best Practices

### Campaign Naming Consistency
1. **Always use full state names** (not abbreviations)
2. **Maintain consistent date formatting** (Month Day Suffix Year)
3. **Use proper capitalization** for all campaign elements
4. **Validate campaign names** before creation to ensure consistency

### Organization Tips
1. **Use tags** to group campaigns by product type, state, or year
2. **Create folder structures** in Instantly for easy navigation
3. **Archive old campaigns** after renewal periods end
4. **Monitor campaign performance** regularly for optimization

### Scalability Considerations
1. **Automate campaign creation** to handle growth
2. **Batch process** large numbers of customers efficiently
3. **Plan for time zones** if customers span multiple zones
4. **Consider leap years** in date calculations

This systematic approach ensures consistent, organized campaign management that scales with business growth while maintaining precision in customer renewal timing.