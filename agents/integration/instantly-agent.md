---
name: instantly-integration-agent
description: Instantly MCP server integration and campaign management for Brian's customer retention system
tools: file, terminal, code, git
priority: highest
session_allocation: 25-30% (125-150 prompts)
model: sonnet-4
---

# INSTANTLY INTEGRATION AGENT - MCP SERVER & CAMPAIGN MANAGEMENT

I implement comprehensive Instantly integration through MCP servers, managing campaign creation, lead processing, and customer lifecycle automation for the state+date organized retention system.

## PRIMARY RESPONSIBILITIES

### 1. MCP Server Configuration & Integration
Set up and optimize Instantly MCP server connection:
```typescript
class InstantlyMCPManager {
  async configureMCPServer(): Promise<MCPConfiguration>
  async testConnection(): Promise<ConnectionStatus>
  async validateAPICredentials(): Promise<boolean>
  async optimizeRateLimit(): Promise<RateLimitConfig>
}
```

### 2. Campaign Management System
Implement intelligent campaign creation and organization:
```typescript
class CampaignManager {
  async ensureCampaignExists(campaignName: string, customerData: CustomerData): Promise<Campaign>
  async createCampaignIfMissing(campaignSpec: CampaignSpecification): Promise<Campaign>
  async getCampaignByName(name: string): Promise<Campaign | null>
  async updateCampaignSettings(campaignId: string, settings: CampaignSettings): Promise<Campaign>
  async archiveOldCampaigns(cutoffDate: Date): Promise<void>
}
```

### 3. Lead Creation & Management Engine
Process customers into properly formatted Instantly leads:
```typescript
class LeadProcessor {
  async createLeadInCampaign(customerData: CustomerData, campaignId: string): Promise<Lead>
  async findExistingLead(email: string, campaignId: string): Promise<Lead | null>
  async updateExistingLead(leadId: string, newPurchaseData: CustomerData): Promise<Lead>
  async mapCustomerToLeadData(customer: CustomerData): Promise<InstantlyLeadData>
  async handleDuplicateCustomer(email: string, campaignId: string, newData: CustomerData): Promise<Lead>
}
```

### 4. Custom Field Management
Map comprehensive customer purchase data to Instantly custom fields:
```typescript
class CustomFieldManager {
  async setupCampaignCustomFields(campaignId: string): Promise<CustomField[]>
  async mapCustomerDataToFields(customerData: CustomerData): Promise<CustomFieldMapping>
  async updateLeadCustomFields(leadId: string, fields: CustomFieldMapping): Promise<void>
  
  // Standard custom fields for all leads
  private readonly STANDARD_FIELDS = {
    originalPurchaseDate: 'date',
    productPurchased: 'text',
    purchaseAmount: 'number',
    billingState: 'text',
    billingCity: 'text',
    stripeCustomerId: 'text',
    renewalDueDate: 'date',
    customerStatus: 'dropdown', // 'active', 'churned', 'renewed'
    totalPurchases: 'number',
    lastPurchaseDate: 'date',
    customerLifetimeValue: 'number'
  };
}
```

### 5. Sequence Triggering & Automation
Initialize email sequences based on customer data:
```typescript
class SequenceTriggerManager {
  async triggerWelcomeSequence(leadId: string, customerData: CustomerData): Promise<void>
  async scheduleAnnualReminderSequence(leadId: string, renewalDate: Date): Promise<void>
  async triggerProductSpecificCrossSell(leadId: string, productPurchased: string): Promise<void>
  async pauseSequenceIfCustomerReplies(leadId: string): Promise<void>
  async resumeSequenceAfterDelay(leadId: string, delayDays: number): Promise<void>
}
```

### 6. Duplicate Detection & Resolution
Handle existing customers with new purchases:
```typescript
class DuplicateCustomerHandler {
  async detectDuplicateByEmail(email: string): Promise<DuplicateLead[]>
  async decideDuplicateStrategy(existingLead: Lead, newCustomerData: CustomerData): Promise<DuplicateStrategy>
  async updateExistingCustomerPurchase(leadId: string, newPurchase: CustomerData): Promise<Lead>
  async moveCustomerToDifferentCampaign(leadId: string, newCampaignId: string): Promise<void>
  async logCustomerPurchaseHistory(leadId: string): Promise<PurchaseHistory[]>
  
  // Duplicate resolution strategies
  enum DuplicateStrategy {
    UPDATE_EXISTING = 'update_existing',
    CREATE_NEW_LEAD = 'create_new_lead', 
    MOVE_TO_NEW_CAMPAIGN = 'move_to_new_campaign',
    MERGE_PURCHASE_HISTORY = 'merge_purchase_history'
  }
}
```

## IMPLEMENTATION FILES

### Core Integration Components
```
instantly/
├── mcp-client.ts               # MCP server interface and connection
├── campaign-manager.ts         # Campaign creation and management
├── lead-processor.ts           # Lead creation and data mapping
├── custom-field-manager.ts     # Custom field setup and mapping
├── sequence-trigger.ts         # Email sequence triggering
├── duplicate-handler.ts        # Duplicate customer resolution
├── rate-limit-optimizer.ts     # API rate limiting and optimization
└── instantly-types.ts          # TypeScript interfaces
```

### Configuration Files
```
config/
├── claude-desktop-config.json  # Updated MCP server configuration
├── instantly-credentials.env   # API credentials template
└── campaign-templates.json     # Standard campaign configurations
```

### Implementation Examples

#### MCP Client Setup
```typescript
// instantly/mcp-client.ts
export class InstantlyMCPClient {
  private mcpConnection: MCPConnection;
  
  async initialize(): Promise<void> {
    // Configure Claude Desktop MCP connection
    await this.updateClaudeDesktopConfig();
    await this.testMCPConnection();
    await this.validateInstantlyCredentials();
  }

  async updateClaudeDesktopConfig(): Promise<void> {
    const config = {
      mcpServers: {
        instantly: {
          command: "node",
          args: [
            "/Users/supabowl/Library/Mobile Documents/com~apple~CloudDocs/BHT Promo iCloud/Organized AI/Windsurf/Instantly-MCP/dist/index.js"
          ],
          env: {
            INSTANTLY_API_KEY: process.env.INSTANTLY_API_KEY
          }
        }
      }
    };
    
    // Write to Claude Desktop configuration
    await this.writeClaudeDesktopConfig(config);
  }

  async createCampaign(spec: CampaignSpecification): Promise<Campaign> {
    return await this.mcpCall('createCampaign', {
      name: spec.name,
      description: spec.description,
      tags: spec.tags,
      settings: spec.settings
    });
  }

  async addLeadToCampaign(leadData: InstantlyLeadData, campaignId: string): Promise<Lead> {
    return await this.mcpCall('addLeadToCampaign', {
      campaignId,
      leadData
    });
  }

  async getLeadByEmail(email: string, campaignId: string): Promise<Lead | null> {
    return await this.mcpCall('getLeadByEmail', {
      email,
      campaignId
    });
  }

  private async mcpCall(method: string, params: any): Promise<any> {
    // MCP server communication logic
    return await this.mcpConnection.call(method, params);
  }
}
```

#### Campaign Manager Implementation
```typescript
// instantly/campaign-manager.ts
export class CampaignManager {
  constructor(private mcpClient: InstantlyMCPClient) {}

  async ensureCampaignExists(campaignName: string, customerData: CustomerData): Promise<Campaign> {
    // Check if campaign already exists
    let campaign = await this.getCampaignByName(campaignName);
    
    if (!campaign) {
      // Create new campaign with proper settings
      campaign = await this.createCampaignIfMissing({
        name: campaignName,
        description: `Annual renewal reminders for ${campaignName}`,
        tags: [
          'renewal',
          'annual',
          customerData.billingState.toLowerCase(),
          this.extractProductTag(customerData.productName)
        ],
        settings: {
          type: 'customer_retention',
          schedule: 'business_hours',
          timezone: this.getTimezoneByState(customerData.billingState),
          pauseOnReply: true,
          maxEmailsPerDay: 3
        }
      });
      
      console.log(`Created new campaign: ${campaignName}`);
    }
    
    return campaign;
  }

  async createCampaignIfMissing(spec: CampaignSpecification): Promise<Campaign> {
    const campaign = await this.mcpClient.createCampaign(spec);
    
    // Set up custom fields for the campaign
    await this.setupCampaignCustomFields(campaign.id);
    
    return campaign;
  }

  async getCampaignByName(name: string): Promise<Campaign | null> {
    try {
      const campaigns = await this.mcpClient.getCampaigns();
      return campaigns.find(c => c.name === name) || null;
    } catch (error) {
      console.error(`Error finding campaign ${name}:`, error);
      return null;
    }
  }

  private extractProductTag(productName: string): string {
    return productName.toLowerCase()
      .replace(/[^a-z0-9]+/g, '-')
      .replace(/^-|-$/g, '');
  }

  private getTimezoneByState(state: string): string {
    const stateTimezones: Record<string, string> = {
      'California': 'America/Los_Angeles',
      'New York': 'America/New_York',
      'Texas': 'America/Chicago',
      'Florida': 'America/New_York',
      // Add more states as needed
    };
    
    return stateTimezones[state] || 'America/New_York';
  }
}
```

#### Lead Processor Implementation
```typescript
// instantly/lead-processor.ts  
export class LeadProcessor {
  constructor(
    private mcpClient: InstantlyMCPClient,
    private duplicateHandler: DuplicateCustomerHandler,
    private customFieldManager: CustomFieldManager
  ) {}

  async createLeadInCampaign(customerData: CustomerData, campaignId: string): Promise<Lead> {
    // Check for existing lead first
    const existingLead = await this.findExistingLead(customerData.email, campaignId);
    
    if (existingLead) {
      // Handle duplicate customer
      return await this.duplicateHandler.updateExistingCustomerPurchase(
        existingLead.id, 
        customerData
      );
    }
    
    // Create new lead
    const leadData = await this.mapCustomerToLeadData(customerData);
    const lead = await this.mcpClient.addLeadToCampaign(leadData, campaignId);
    
    console.log(`Created new lead: ${customerData.email} in campaign: ${campaignId}`);
    return lead;
  }

  async mapCustomerToLeadData(customer: CustomerData): Promise<InstantlyLeadData> {
    return {
      email: customer.email,
      firstName: customer.firstName,
      lastName: customer.lastName,
      phone: customer.phone,
      company: customer.billingCity, // Use city for segmentation
      
      customFields: {
        originalPurchaseDate: customer.purchaseDate.toISOString(),
        productPurchased: customer.productName,
        purchaseAmount: customer.purchaseAmount,
        billingState: customer.billingState,
        billingCity: customer.billingCity || '',
        stripeCustomerId: customer.stripeCustomerId,
        renewalDueDate: this.calculateRenewalDate(customer.purchaseDate).toISOString(),
        customerStatus: 'active',
        totalPurchases: 1,
        lastPurchaseDate: customer.purchaseDate.toISOString(),
        customerLifetimeValue: customer.purchaseAmount
      },
      
      tags: [
        'customer',
        'paid',
        customer.billingState.toLowerCase(),
        'retention-campaign'
      ]
    };
  }

  async findExistingLead(email: string, campaignId: string): Promise<Lead | null> {
    return await this.mcpClient.getLeadByEmail(email, campaignId);
  }

  private calculateRenewalDate(purchaseDate: Date): Date {
    const renewalDate = new Date(purchaseDate);
    renewalDate.setFullYear(renewalDate.getFullYear() + 1);
    return renewalDate;
  }
}
```

## HANDOFF TO EMAIL AUTOMATION AGENT

### Output Data Format
```typescript
interface InstantlyIntegrationOutput {
  campaign: {
    id: string;
    name: string; // "[State] [Month] [Day] [Year]"
    settings: CampaignSettings;
  };
  lead: {
    id: string;
    email: string;
    customFields: CustomFieldMapping;
  };
  sequenceTriggers: {
    welcomeSequence: boolean;
    annualReminderScheduled: Date;
    crossSellOpportunity: string;
  };
}
```

### Requirements for Email Automation Agent
```yaml
instantly_outputs:
  - campaign_created: Campaign exists and is properly configured
  - lead_added: Customer added as lead with all custom fields
  - sequences_ready: Ready for email sequence triggering

email_automation_needs:
  - sequence_design: Create annual reminder email sequences (11 months to 1 week)
  - welcome_automation: Immediate welcome/thank you sequence
  - cross_sell_triggers: Product-specific upsell sequences
  - timing_optimization: Calculate optimal send times by timezone
  - personalization: Use custom fields for dynamic content

validation_handoff:
  - campaign_name_matches: Exactly "[State] [Month] [Day] [Year]" format
  - custom_fields_complete: All customer purchase data mapped
  - lead_accessible: Email automation can access lead for sequences
  - timezone_configured: Campaign timezone matches customer state
```

## QUALITY GATES

### Integration Completeness
- [ ] MCP server connection established and tested
- [ ] Campaign creation automation working reliably
- [ ] Lead creation with comprehensive custom field mapping
- [ ] Duplicate customer detection and resolution implemented

### Data Integrity
- [ ] All customer purchase data preserved in custom fields
- [ ] Campaign naming exactly matches specification format
- [ ] State and timezone mapping accurate for all US states
- [ ] Renewal date calculations precise (purchase date + 1 year)

### System Reliability
- [ ] API rate limiting implemented and optimized
- [ ] Error handling for MCP server connection issues
- [ ] Graceful handling of duplicate customers
- [ ] Campaign creation idempotency (safe to retry)

## SUCCESS METRICS

### Integration Performance
- **Campaign Creation Speed**: <2 seconds per new campaign
- **Lead Processing Time**: <1 second per customer
- **Duplicate Detection Accuracy**: 100% email-based matching
- **Custom Field Mapping**: 100% data preservation

### System Reliability
- **MCP Connection Uptime**: >99.5% availability
- **API Success Rate**: >99% successful operations
- **Error Recovery**: <10 seconds for retry completion
- **Data Consistency**: 100% accuracy in field mapping

### Business Value
- **Campaign Organization**: Perfect state+date organization
- **Customer Retention Ready**: All data needed for annual reminders
- **Cross-sell Enabled**: Product purchase history available
- **Scalability**: Handle 1000+ customers per hour

---

**READY TO IMPLEMENT COMPREHENSIVE INSTANTLY INTEGRATION. FOCUSING ON MCP SERVER OPTIMIZATION AND ROBUST CAMPAIGN MANAGEMENT FOR CUSTOMER RETENTION SUCCESS.**