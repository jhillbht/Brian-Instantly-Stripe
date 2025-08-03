# MCP Server Setup Guide - Instantly & Stripe

## Prerequisites

### System Requirements
- **Machine**: Macbook M1 Pro (supabowl)
- **Node.js**: Version 18 or higher
- **Claude Desktop**: Latest version installed
- **Cursor**: Latest version installed
- **Git**: For cloning repositories

### API Keys Required
- **Instantly API Key**: From your Instantly account settings
- **Stripe API Keys**: Secret key and publishable key from Stripe dashboard

## Instantly MCP Server Setup

### 1. Clone and Install Instantly MCP Server

```bash
# Navigate to your workspace
cd "/Users/supabowl/Library/Mobile Documents/com~apple~CloudDocs/BHT Promo iCloud/Organized AI/Windsurf"

# Clone the repository
git clone https://github.com/bcharleson/Instantly-MCP.git
cd Instantly-MCP

# Install dependencies
npm install

# Build the server
npm run build
```

### 2. Configure Environment Variables

Create `.env` file in the Instantly-MCP directory:

```env
INSTANTLY_API_KEY=your_instantly_api_key_here
INSTANTLY_BASE_URL=https://api.instantly.ai
```

### 3. Test Instantly MCP Server

```bash
# Test the server locally
npm start

# Verify it's running (should show MCP server info)
curl http://localhost:3000/health
```

## Stripe MCP Server Setup

### 1. Install Stripe MCP Server

```bash
# Navigate back to workspace root
cd "/Users/supabowl/Library/Mobile Documents/com~apple~CloudDocs/BHT Promo iCloud/Organized AI/Windsurf"

# Install Stripe CLI (if not already installed)
brew install stripe/stripe-cli/stripe

# Login to Stripe
stripe login

# Clone or create Stripe MCP server directory
mkdir stripe-mcp && cd stripe-mcp

# Initialize npm project
npm init -y

# Install required dependencies
npm install @stripe/stripe-js stripe @types/node typescript ts-node
```

### 2. Create Stripe MCP Server

Create `stripe-mcp-server.ts`:

```typescript
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

// MCP server implementation for Stripe
// (This will be expanded based on Stripe's MCP documentation)

export class StripeMCPServer {
  constructor(private stripe: Stripe) {}

  async createCustomer(email: string, name?: string) {
    return await this.stripe.customers.create({
      email,
      name,
    });
  }

  async createPaymentLink(priceId: string, quantity = 1) {
    return await this.stripe.paymentLinks.create({
      line_items: [{ price: priceId, quantity }],
    });
  }

  async getCustomer(customerId: string) {
    return await this.stripe.customers.retrieve(customerId);
  }

  async listPayments(customerId?: string) {
    const params: Stripe.PaymentIntentListParams = {};
    if (customerId) {
      params.customer = customerId;
    }
    return await this.stripe.paymentIntents.list(params);
  }
}
```

### 3. Configure Stripe Environment

Create `.env` file in the stripe-mcp directory:

```env
STRIPE_SECRET_KEY=sk_test_your_secret_key_here
STRIPE_PUBLISHABLE_KEY=pk_test_your_publishable_key_here
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret_here
```

## Claude Desktop Configuration

### 1. Update Claude Desktop MCP Configuration

Edit the Claude Desktop configuration file:

**macOS Location**: `~/Library/Application Support/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "instantly": {
      "command": "node",
      "args": [
        "/Users/supabowl/Library/Mobile Documents/com~apple~CloudDocs/BHT Promo iCloud/Organized AI/Windsurf/Instantly-MCP/dist/index.js"
      ],
      "env": {
        "INSTANTLY_API_KEY": "your_instantly_api_key_here"
      }
    },
    "stripe": {
      "command": "node",
      "args": [
        "/Users/supabowl/Library/Mobile Documents/com~apple~CloudDocs/BHT Promo iCloud/Organized AI/Windsurf/stripe-mcp/dist/index.js"
      ],
      "env": {
        "STRIPE_SECRET_KEY": "sk_test_your_secret_key_here",
        "STRIPE_PUBLISHABLE_KEY": "pk_test_your_publishable_key_here"
      }
    }
  }
}
```

### 2. Restart Claude Desktop

1. Quit Claude Desktop completely
2. Restart the application
3. Verify MCP servers are loaded (check status in Claude Desktop)

## Cursor Configuration

### 1. Cursor Extensions Setup

Install relevant extensions:
- TypeScript and JavaScript support
- REST Client (for API testing)
- Git integration

### 2. Workspace Configuration

Create `.vscode/settings.json` in your workspace:

```json
{
  "typescript.preferences.useAliasesForRenames": false,
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  },
  "files.associations": {
    "*.json": "jsonc"
  }
}
```

### 3. Create Development Scripts

Create `package.json` in your workspace root:

```json
{
  "name": "instantly-stripe-integration",
  "version": "1.0.0",
  "scripts": {
    "start:instantly": "cd Instantly-MCP && npm start",
    "start:stripe": "cd stripe-mcp && npm start",
    "build:all": "cd Instantly-MCP && npm run build && cd ../stripe-mcp && npm run build",
    "test:instantly": "cd Instantly-MCP && npm test",
    "test:stripe": "cd stripe-mcp && npm test"
  }
}
```

## Testing and Verification

### 1. Test Instantly MCP Functions

In Claude Desktop, try these commands:
```
Can you list my Instantly campaigns?
Create a new email sequence in Instantly
Show me my lead statistics from Instantly
```

### 2. Test Stripe MCP Functions

In Claude Desktop, try these commands:
```
Create a new Stripe customer with email test@example.com
Generate a payment link for product X
Show me recent payments in Stripe
List all customers in Stripe
```

### 3. Test Integration

```
Create a customer in Stripe for this Instantly lead: [lead email]
Generate a payment link and add it to an Instantly email template
Track which Instantly campaigns have the highest Stripe conversion rates
```

## Troubleshooting

### Common Issues

1. **MCP Server Not Loading**
   - Check file paths in configuration
   - Verify environment variables are set
   - Check server logs for errors

2. **API Authentication Errors**
   - Verify API keys are correct and active
   - Check API key permissions in respective platforms
   - Ensure environment variables are properly loaded

3. **Connection Timeouts**
   - Check internet connection
   - Verify API endpoints are accessible
   - Increase timeout values if needed

### Debug Commands

```bash
# Check if servers are running
ps aux | grep node

# View server logs
tail -f Instantly-MCP/logs/server.log
tail -f stripe-mcp/logs/server.log

# Test API connections directly
curl -H "Authorization: Bearer $INSTANTLY_API_KEY" https://api.instantly.ai/campaigns
curl -H "Authorization: Bearer $STRIPE_SECRET_KEY" https://api.stripe.com/v1/customers
```

## Security Best Practices

1. **Environment Variables**: Never commit API keys to version control
2. **Access Control**: Use least-privilege API keys
3. **Monitoring**: Set up logging for all API calls
4. **Rate Limiting**: Implement proper rate limiting to avoid API limits
5. **Error Handling**: Graceful error handling for API failures

## Next Steps

1. Complete MCP server installations
2. Test basic functionality with both services
3. Implement webhook handlers for real-time integration
4. Set up monitoring and logging
5. Create specific workflow implementations
6. Test with small dataset before full deployment