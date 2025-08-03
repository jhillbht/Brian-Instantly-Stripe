# Stripe Webhook Implementation Guide

## Overview

This guide provides the technical implementation for capturing Stripe purchase events and automatically creating Instantly leads in the correct campaigns based on customer location and purchase timing.

## Webhook Setup Requirements

### Stripe Webhook Configuration

1. **Login to Stripe Dashboard** → Developers → Webhooks
2. **Create New Endpoint** with your server URL: `https://yourdomain.com/stripe-webhook`
3. **Select Events to Listen For**:
   - `checkout.session.completed` (for Stripe Checkout purchases)
   - `payment_intent.succeeded` (for Payment Intent purchases)
   - `invoice.payment_succeeded` (for subscription payments)

### Endpoint Security
```javascript
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const endpointSecret = process.env.STRIPE_WEBHOOK_SECRET;

// Verify webhook signature
function verifyStripeSignature(payload, signature) {
  try {
    return stripe.webhooks.constructEvent(payload, signature, endpointSecret);
  } catch (err) {
    console.error('Webhook signature verification failed:', err.message);
    throw err;
  }
}
```

## Data Extraction from Stripe Events

### Checkout Session Completed Event

```javascript
async function processCheckoutSession(session) {
  // Extract customer information
  const customer = await stripe.customers.retrieve(session.customer);
  const lineItems = await stripe.checkout.sessions.listLineItems(session.id, {
    expand: ['data.price.product']
  });

  const customerData = {
    email: customer.email || session.customer_email,
    firstName: parseFirstName(customer.name || session.customer_details?.name),
    lastName: parseLastName(customer.name || session.customer_details?.name),
    fullName: customer.name || session.customer_details?.name,
    phone: customer.phone || session.customer_details?.phone,
    
    // Purchase details
    productName: lineItems.data[0]?.price?.product?.name,
    productDescription: lineItems.data[0]?.description,
    purchaseAmount: session.amount_total / 100,
    currency: session.currency.toUpperCase(),
    purchaseDate: new Date(session.created * 1000),
    
    // Billing address
    billingAddress: session.customer_details?.address || customer.address,
    billingState: (session.customer_details?.address?.state || customer.address?.state),
    billingCity: (session.customer_details?.address?.city || customer.address?.city),
    billingCountry: (session.customer_details?.address?.country || customer.address?.country),
    
    // Stripe IDs for reference
    stripeCustomerId: customer.id,
    stripeSessionId: session.id,
    stripePaymentIntentId: session.payment_intent
  };

  return customerData;
}
```

### Payment Intent Succeeded Event

```javascript
async function processPaymentIntent(paymentIntent) {
  const customer = await stripe.customers.retrieve(paymentIntent.customer);
  
  const customerData = {
    email: customer.email,
    firstName: parseFirstName(customer.name),
    lastName: parseLastName(customer.name),
    fullName: customer.name,
    phone: customer.phone,
    
    // Purchase details
    purchaseAmount: paymentIntent.amount / 100,
    currency: paymentIntent.currency.toUpperCase(),
    purchaseDate: new Date(paymentIntent.created * 1000),
    
    // Billing address
    billingAddress: paymentIntent.charges?.data[0]?.billing_details?.address || customer.address,
    billingState: paymentIntent.charges?.data[0]?.billing_details?.address?.state || customer.address?.state,
    billingCity: paymentIntent.charges?.data[0]?.billing_details?.address?.city || customer.address?.city,
    
    // Metadata (if product info stored here)
    productName: paymentIntent.metadata?.product_name,
    
    // Stripe IDs
    stripeCustomerId: customer.id,
    stripePaymentIntentId: paymentIntent.id
  };

  return customerData;
}
```

## Name Parsing Functions

```javascript
function parseFirstName(fullName) {
  if (!fullName) return '';
  return fullName.trim().split(' ')[0] || '';
}

function parseLastName(fullName) {
  if (!fullName) return '';
  const nameParts = fullName.trim().split(' ');
  return nameParts.length > 1 ? nameParts.slice(1).join(' ') : '';
}
```

## Campaign Name Generation

```javascript
class CampaignNameGenerator {
  constructor() {
    this.months = [
      'January', 'February', 'March', 'April', 'May', 'June',
      'July', 'August', 'September', 'October', 'November', 'December'
    ];
  }

  generate(state, purchaseDate) {
    // Calculate renewal date (one year later)
    const renewalDate = new Date(purchaseDate);
    renewalDate.setFullYear(renewalDate.getFullYear() + 1);
    
    const year = renewalDate.getFullYear();
    const month = this.months[renewalDate.getMonth()];
    const day = this.addOrdinalSuffix(renewalDate.getDate());
    
    return `${state} ${month} ${day} ${year}`;
  }

  addOrdinalSuffix(day) {
    if (day >= 11 && day <= 13) return `${day}th`;
    
    switch (day % 10) {
      case 1: return `${day}st`;
      case 2: return `${day}nd`;
      case 3: return `${day}rd`;
      default: return `${day}th`;
    }
  }
}
```

## Instantly Integration Functions

### Campaign Management

```javascript
const instantly = require('./instantly-client'); // Your Instantly API client

async function ensureCampaignExists(campaignName, customerData) {
  try {
    // Check if campaign already exists
    const existingCampaign = await instantly.getCampaign(campaignName);
    
    if (existingCampaign) {
      return existingCampaign;
    }
    
    // Create new campaign
    const newCampaign = await instantly.createCampaign({
      name: campaignName,
      description: `Annual renewal reminders for ${campaignName}`,
      tags: [
        'renewal',
        'annual',
        customerData.billingState?.toLowerCase(),
        customerData.productName?.toLowerCase().replace(/\s+/g, '-')
      ],
      settings: {
        type: 'customer_retention',
        schedule: 'business_hours',
        timezone: getTimezoneByState(customerData.billingState)
      }
    });
    
    console.log(`Created new campaign: ${campaignName}`);
    return newCampaign;
    
  } catch (error) {
    console.error(`Error managing campaign ${campaignName}:`, error);
    throw error;
  }
}
```

### Lead Creation

```javascript
async function addLeadToCampaign(customerData, campaignName) {
  try {
    // Ensure campaign exists
    const campaign = await ensureCampaignExists(campaignName, customerData);
    
    // Check if lead already exists
    const existingLead = await instantly.findLead(customerData.email, campaign.id);
    
    if (existingLead) {
      // Update existing lead with new purchase info
      await instantly.updateLead(existingLead.id, {
        customFields: {
          lastPurchaseDate: customerData.purchaseDate.toISOString(),
          lastPurchaseAmount: customerData.purchaseAmount,
          lastProductPurchased: customerData.productName,
          totalPurchases: (existingLead.customFields?.totalPurchases || 0) + 1
        }
      });
      
      console.log(`Updated existing lead: ${customerData.email}`);
      return existingLead;
    }
    
    // Create new lead
    const newLead = await instantly.createLead({
      email: customerData.email,
      firstName: customerData.firstName,
      lastName: customerData.lastName,
      phone: customerData.phone,
      company: customerData.billingCity, // Store city as company for segmentation
      
      customFields: {
        originalPurchaseDate: customerData.purchaseDate.toISOString(),
        productPurchased: customerData.productName,
        purchaseAmount: customerData.purchaseAmount,
        billingState: customerData.billingState,
        billingCity: customerData.billingCity,
        stripeCustomerId: customerData.stripeCustomerId,
        renewalDueDate: new Date(customerData.purchaseDate.getFullYear() + 1, 
                               customerData.purchaseDate.getMonth(), 
                               customerData.purchaseDate.getDate()).toISOString(),
        customerStatus: 'active',
        totalPurchases: 1
      },
      
      campaignId: campaign.id,
      tags: ['customer', 'paid', customerData.billingState?.toLowerCase()]
    });
    
    console.log(`Created new lead: ${customerData.email} in campaign: ${campaignName}`);
    return newLead;
    
  } catch (error) {
    console.error(`Error adding lead to campaign:`, error);
    throw error;
  }
}
```

## Complete Webhook Handler

```javascript
const express = require('express');
const app = express();

// Middleware for raw body (required for Stripe webhook verification)
app.use('/stripe-webhook', express.raw({ type: 'application/json' }));

app.post('/stripe-webhook', async (req, res) => {
  const sig = req.headers['stripe-signature'];
  let event;

  try {
    // Verify webhook signature
    event = verifyStripeSignature(req.body, sig);
  } catch (err) {
    console.error('Webhook signature verification failed:', err.message);
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  try {
    let customerData;
    
    // Handle different event types
    switch (event.type) {
      case 'checkout.session.completed':
        customerData = await processCheckoutSession(event.data.object);
        break;
        
      case 'payment_intent.succeeded':
        customerData = await processPaymentIntent(event.data.object);
        break;
        
      case 'invoice.payment_succeeded':
        // Handle subscription payments if needed
        customerData = await processInvoicePayment(event.data.object);
        break;
        
      default:
        console.log(`Unhandled event type: ${event.type}`);
        return res.status(200).send('OK');
    }

    // Skip if no billing state (international customers, etc.)
    if (!customerData.billingState) {
      console.log(`Skipping customer without billing state: ${customerData.email}`);
      return res.status(200).send('OK');
    }

    // Generate campaign name
    const campaignNameGenerator = new CampaignNameGenerator();
    const campaignName = campaignNameGenerator.generate(
      customerData.billingState,
      customerData.purchaseDate
    );

    // Add to Instantly campaign
    await addLeadToCampaign(customerData, campaignName);
    
    console.log(`Successfully processed customer: ${customerData.email} → ${campaignName}`);
    
  } catch (error) {
    console.error('Error processing webhook:', error);
    return res.status(500).send('Internal Server Error');
  }

  res.status(200).send('OK');
});
```

## Error Handling & Retry Logic

```javascript
async function processWithRetry(fn, maxRetries = 3, delay = 1000) {
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      console.error(`Attempt ${attempt} failed:`, error.message);
      
      if (attempt < maxRetries) {
        console.log(`Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        delay *= 2; // Exponential backoff
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
}

// Usage in webhook handler
await processWithRetry(async () => {
  await addLeadToCampaign(customerData, campaignName);
});
```

## State Mapping & Validation

```javascript
const US_STATES = {
  'AL': 'Alabama', 'AK': 'Alaska', 'AZ': 'Arizona', 'AR': 'Arkansas',
  'CA': 'California', 'CO': 'Colorado', 'CT': 'Connecticut', 'DE': 'Delaware',
  'FL': 'Florida', 'GA': 'Georgia', 'HI': 'Hawaii', 'ID': 'Idaho',
  'IL': 'Illinois', 'IN': 'Indiana', 'IA': 'Iowa', 'KS': 'Kansas',
  'KY': 'Kentucky', 'LA': 'Louisiana', 'ME': 'Maine', 'MD': 'Maryland',
  'MA': 'Massachusetts', 'MI': 'Michigan', 'MN': 'Minnesota', 'MS': 'Mississippi',
  'MO': 'Missouri', 'MT': 'Montana', 'NE': 'Nebraska', 'NV': 'Nevada',
  'NH': 'New Hampshire', 'NJ': 'New Jersey', 'NM': 'New Mexico', 'NY': 'New York',
  'NC': 'North Carolina', 'ND': 'North Dakota', 'OH': 'Ohio', 'OK': 'Oklahoma',
  'OR': 'Oregon', 'PA': 'Pennsylvania', 'RI': 'Rhode Island', 'SC': 'South Carolina',
  'SD': 'South Dakota', 'TN': 'Tennessee', 'TX': 'Texas', 'UT': 'Utah',
  'VT': 'Vermont', 'VA': 'Virginia', 'WA': 'Washington', 'WV': 'West Virginia',
  'WI': 'Wisconsin', 'WY': 'Wyoming', 'DC': 'District of Columbia'
};

function normalizeStateName(stateInput) {
  if (!stateInput) return null;
  
  const state = stateInput.toString().trim();
  
  // If it's already a full state name, return it
  if (Object.values(US_STATES).includes(state)) {
    return state;
  }
  
  // If it's an abbreviation, convert to full name
  const upperState = state.toUpperCase();
  if (US_STATES[upperState]) {
    return US_STATES[upperState];
  }
  
  // Try to find partial matches
  const matches = Object.values(US_STATES).filter(fullName => 
    fullName.toLowerCase().includes(state.toLowerCase())
  );
  
  if (matches.length === 1) {
    return matches[0];
  }
  
  console.warn(`Could not normalize state: ${stateInput}`);
  return null;
}
```

## Testing & Debugging

### Test Webhook Locally

```bash
# Install Stripe CLI
brew install stripe/stripe-cli/stripe

# Login to Stripe
stripe login

# Forward webhooks to local development server
stripe listen --forward-to localhost:3000/stripe-webhook

# Trigger test webhook
stripe trigger checkout.session.completed
```

### Debug Webhook Data

```javascript
app.post('/stripe-webhook-debug', express.raw({ type: 'application/json' }), (req, res) => {
  const sig = req.headers['stripe-signature'];
  
  try {
    const event = verifyStripeSignature(req.body, sig);
    
    // Log the complete event for debugging
    console.log('=== STRIPE WEBHOOK DEBUG ===');
    console.log('Event Type:', event.type);
    console.log('Event Data:', JSON.stringify(event.data.object, null, 2));
    console.log('===========================');
    
    res.status(200).send('Debug logged');
  } catch (err) {
    console.error('Debug webhook error:', err);
    res.status(400).send('Error');
  }
});
```

This implementation provides a robust foundation for capturing Stripe purchases and automatically organizing customers into Instantly campaigns for annual renewal reminders and cross-selling opportunities.