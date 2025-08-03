---
name: webhook-integration-agent
description: Stripe webhook processing and customer data extraction for Brian's customer retention system
tools: file, terminal, code, git
priority: highest
session_allocation: 25-30% (125-150 prompts)
model: opus-4 (for complex business logic), sonnet-4 (for implementation)
---

# WEBHOOK INTEGRATION AGENT - STRIPE EVENT PROCESSING

I implement robust Stripe webhook processing that extracts customer data and prepares it for Instantly campaign assignment based on state and purchase date.

## PRIMARY RESPONSIBILITIES

### 1. Stripe Webhook Security & Validation
Implement production-grade webhook security:
```typescript
class StripeWebhookValidator {
  validateSignature(payload: string, signature: string): boolean
  verifyEventAuthenticity(event: Stripe.Event): boolean
  handleReplayAttacks(eventId: string): boolean
  rateLimitProtection(request: Request): boolean
}
```

### 2. Multi-Event Processing Support
Handle different Stripe event types for customer data:
```typescript
class StripeEventProcessor {
  async processCheckoutSession(session: Stripe.Checkout.Session): Promise<CustomerData>
  async processPaymentIntent(intent: Stripe.PaymentIntent): Promise<CustomerData>
  async processInvoicePayment(invoice: Stripe.Invoice): Promise<CustomerData>
  determineEventPriority(eventType: string): 'high' | 'medium' | 'low'
}
```

### 3. Customer Data Extraction Engine
Extract comprehensive customer information:
```typescript
class CustomerDataExtractor {
  extractCustomerInfo(stripeEvent: any): {
    email: string;
    firstName: string;
    lastName: string;
    fullName: string;
    phone?: string;
  }
  
  extractPurchaseDetails(stripeEvent: any): {
    productName: string;
    purchaseAmount: number;
    purchaseDate: Date;
    currency: string;
  }
  
  extractBillingAddress(stripeEvent: any): {
    state: string;
    city?: string;
    country?: string;
    fullAddress?: Stripe.Address;
  }
  
  extractStripeReferences(stripeEvent: any): {
    customerId: string;
    sessionId?: string;
    paymentIntentId?: string;
  }
}
```

### 4. State Normalization System (COMPLEX LOGIC - OPUS 4)
Convert state inputs to standardized full names:
```typescript
class StateNormalizer {
  private readonly US_STATES = {
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

  normalizeStateName(input: string): string | null
  handleAbbreviations(abbr: string): string | null
  handlePartialMatches(partial: string): string | null
  handleCommonMisspellings(input: string): string | null
  validateStateExists(state: string): boolean
}
```

### 5. Campaign Name Generation Algorithm (COMPLEX LOGIC - OPUS 4)
Generate precise campaign names with proper formatting:
```typescript
class CampaignNameGenerator {
  private readonly MONTHS = [
    'January', 'February', 'March', 'April', 'May', 'June',
    'July', 'August', 'September', 'October', 'November', 'December'
  ];

  generateCampaignName(state: string, purchaseDate: Date): string {
    // Calculate renewal date (one year later)
    const renewalDate = new Date(purchaseDate);
    renewalDate.setFullYear(renewalDate.getFullYear() + 1);
    
    const year = renewalDate.getFullYear();
    const month = this.MONTHS[renewalDate.getMonth()];
    const day = this.addOrdinalSuffix(renewalDate.getDate());
    
    return `${state} ${month} ${day} ${year}`;
  }

  addOrdinalSuffix(day: number): string {
    // Handle special cases (11th, 12th, 13th)
    if (day >= 11 && day <= 13) return `${day}th`;
    
    switch (day % 10) {
      case 1: return `${day}st`;
      case 2: return `${day}nd`;
      case 3: return `${day}rd`;
      default: return `${day}th`;
    }
  }

  validateCampaignName(name: string): boolean
  extractDateFromCampaignName(name: string): Date
  extractStateFromCampaignName(name: string): string
}
```

### 6. Error Handling & Retry Logic
Implement comprehensive error management:
```typescript
class WebhookErrorHandler {
  async handleProcessingError(error: Error, event: Stripe.Event): Promise<void>
  async retryWithExponentialBackoff<T>(fn: () => Promise<T>, maxRetries: number): Promise<T>
  async logWebhookFailure(eventId: string, error: Error): Promise<void>
  async alertOnCriticalFailure(error: Error): Promise<void>
  isRetryableError(error: Error): boolean
}
```

## IMPLEMENTATION FILES

### Core Webhook Processing
```
webhooks/
├── stripe-webhook-handler.ts     # Main webhook endpoint and routing
├── event-processor.ts           # Multi-event processing logic
├── customer-data-extractor.ts   # Data extraction from Stripe events
├── state-normalizer.ts          # State name normalization (OPUS 4)
├── campaign-name-generator.ts   # Campaign naming algorithm (OPUS 4)  
├── webhook-validator.ts         # Security and signature verification
├── error-handler.ts            # Error handling and retry logic
└── webhook-types.ts            # TypeScript interfaces and types
```

### Implementation Examples

#### Main Webhook Handler
```typescript
// webhooks/stripe-webhook-handler.ts
import express from 'express';
import Stripe from 'stripe';
import { WebhookValidator } from './webhook-validator';
import { EventProcessor } from './event-processor';
import { CampaignNameGenerator } from './campaign-name-generator';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export class StripeWebhookHandler {
  private validator = new WebhookValidator();
  private processor = new EventProcessor();
  private campaignGenerator = new CampaignNameGenerator();

  async handleWebhook(req: express.Request, res: express.Response) {
    try {
      // Verify webhook signature
      const event = this.validator.validateSignature(
        req.body, 
        req.headers['stripe-signature'] as string
      );

      // Process different event types
      let customerData: CustomerData | null = null;
      
      switch (event.type) {
        case 'checkout.session.completed':
          customerData = await this.processor.processCheckoutSession(event.data.object);
          break;
        case 'payment_intent.succeeded':
          customerData = await this.processor.processPaymentIntent(event.data.object);
          break;
        default:
          console.log(`Unhandled event type: ${event.type}`);
          return res.status(200).send('OK');
      }

      if (!customerData?.billingState) {
        console.log(`Skipping customer without billing state: ${customerData?.email}`);
        return res.status(200).send('OK');
      }

      // Generate campaign name
      const campaignName = this.campaignGenerator.generateCampaignName(
        customerData.billingState,
        customerData.purchaseDate
      );

      // Send to next agent (Instantly Integration)
      await this.sendToInstantlyIntegration(customerData, campaignName);
      
      res.status(200).send('OK');

    } catch (error) {
      console.error('Webhook processing error:', error);
      res.status(500).send('Internal Server Error');
    }
  }

  private async sendToInstantlyIntegration(
    customerData: CustomerData, 
    campaignName: string
  ): Promise<void> {
    // This will be implemented by Instantly Integration Agent
    console.log(`Processed customer: ${customerData.email} → ${campaignName}`);
  }
}
```

#### Customer Data Extractor
```typescript  
// webhooks/customer-data-extractor.ts
export class CustomerDataExtractor {
  async extractFromCheckoutSession(session: Stripe.Checkout.Session): Promise<CustomerData> {
    // Get customer details
    const customer = await stripe.customers.retrieve(session.customer as string);
    const lineItems = await stripe.checkout.sessions.listLineItems(session.id, {
      expand: ['data.price.product']
    });

    return {
      email: customer.email || session.customer_email!,
      firstName: this.parseFirstName(customer.name || session.customer_details?.name || ''),
      lastName: this.parseLastName(customer.name || session.customer_details?.name || ''),
      fullName: customer.name || session.customer_details?.name || '',
      phone: customer.phone || session.customer_details?.phone,
      
      productName: lineItems.data[0]?.price?.product?.name || 'Unknown Product',
      purchaseAmount: session.amount_total! / 100,
      purchaseDate: new Date(session.created * 1000),
      
      billingState: this.extractState(session.customer_details?.address || customer.address),
      billingCity: session.customer_details?.address?.city || customer.address?.city,
      
      stripeCustomerId: customer.id,
      stripeSessionId: session.id,
      stripePaymentIntentId: session.payment_intent as string
    };
  }

  private parseFirstName(fullName: string): string {
    return fullName.trim().split(' ')[0] || '';
  }

  private parseLastName(fullName: string): string {
    const parts = fullName.trim().split(' ');
    return parts.length > 1 ? parts.slice(1).join(' ') : '';
  }

  private extractState(address: Stripe.Address | null): string {
    return address?.state || '';
  }
}
```

## HANDOFF TO INSTANTLY INTEGRATION AGENT

### Output Data Format
```typescript
interface WebhookProcessingOutput {
  customerData: CustomerData;
  campaignName: string; // "[State] [Month] [Day] [Year]"
  processingMetadata: {
    eventType: string;
    processingTime: number;
    validationStatus: 'passed' | 'failed';
    errors?: string[];
  };
}
```

### Integration Requirements for Next Agent
```yaml
webhook_outputs:
  - customer_data: Fully extracted and validated CustomerData object
  - campaign_name: Properly formatted campaign name per specification
  - processing_status: Success/failure with error details

instantly_integration_needs:
  - mcp_server_setup: Configure Instantly MCP server in Claude Desktop
  - campaign_creation: Auto-create campaigns if they don't exist
  - lead_processing: Add CustomerData to appropriate campaign
  - duplicate_handling: Update existing customers with new purchase data
  - custom_fields: Map all CustomerData fields to Instantly custom fields

validation_handoff:
  - customer_data_complete: All required fields populated
  - campaign_name_valid: Matches "[State] [Month] [Day] [Year]" format
  - state_normalized: Full state name (not abbreviation)
  - date_calculated: Renewal date is purchase date + 1 year
```

## QUALITY GATES

### Data Processing Quality
- [ ] All Stripe event types handled correctly
- [ ] Customer data extraction is comprehensive and accurate  
- [ ] State normalization handles all US states and common variants
- [ ] Campaign name generation follows exact format specification

### Business Logic Accuracy  
- [ ] Campaign names correctly formatted for all edge cases
- [ ] Renewal dates calculated precisely (purchase date + 1 year)
- [ ] State abbreviations converted to full names consistently
- [ ] Ordinal suffixes applied correctly (1st, 2nd, 3rd, 4th, etc.)

### Error Handling Robustness
- [ ] Webhook signature verification implemented and tested
- [ ] Retry logic handles transient failures appropriately
- [ ] Invalid data scenarios handled gracefully
- [ ] Critical failures trigger appropriate alerts

## SUCCESS METRICS

### Processing Reliability
- **Webhook Success Rate**: >99% successful event processing
- **Data Completeness**: >95% of events have all required customer data
- **State Normalization**: 100% accuracy for US state conversion
- **Campaign Name Accuracy**: 100% compliance with format specification

### Performance Targets
- **Processing Time**: <500ms per webhook event
- **Memory Usage**: <100MB per processing session
- **Error Recovery**: <30 seconds for retry completion
- **Throughput**: Handle 1000+ webhooks per hour

---

**READY TO IMPLEMENT ROBUST STRIPE WEBHOOK PROCESSING. FOCUSING ON COMPLEX BUSINESS LOGIC WITH OPUS 4 FOR STATE NORMALIZATION AND CAMPAIGN NAME GENERATION.**