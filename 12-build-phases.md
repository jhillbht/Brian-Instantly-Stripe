# Build Phases - Brian Instantly Stripe Integration

**Plan**: $200 Max Plan (10x Pro usage)  
**Resource Planning**: Token/Session/Rate Limit Based  
**Goal**: Customer retention automation system with optimal resource usage  
**Project**: Stripe → Instantly integration for annual renewal reminders

---

## 📊 Resource Summary - $200 Max Plan

### Web Interface (Claude Chat)
- **Session Capacity**: 450 messages per 5-hour session
- **Daily Capacity**: ~2,160 messages (4.8 sessions × 450)
- **Weekly Capacity**: ~15,120 messages
- **Reset Cycle**: Every 5 hours

### Claude Code
- **Weekly Sonnet 4**: 280-560 hours (conservative: 280h)
- **Weekly Opus 4**: 30-70 hours (conservative: 30h) 
- **Session Capacity**: 100-400 prompts per 5 hours
- **Daily Sonnet**: ~40 hours (280h ÷ 7 days)
- **Reset Cycle**: Weekly (every 7 days)

---

## Phase 1: Architecture Foundation (Steps 1-10)
**Model Strategy**: Sonnet 4 for setup, Opus 4 for webhook design complexity
**Resource Estimate**: 
- Sonnet 4: 4-5 hours (Steps 1-3, 7-10: setup, testing, validation)
- Opus 4: 2-3 hours (Steps 4-6: webhook architecture and state logic)
- Total: 6-8 hours
**Claude Code Prompts**: 30-40 prompts + 35 web messages  
**Goal**: Webhook architecture and MCP server setup

### Resource Breakdown by Step

#### Step 1-3: Project Setup & Environment
**Claude Code Prompts**: 8-12 prompts  
**Sonnet 4 Time**: ~1.5 hours  
**Web Messages**: 10 messages (environment research)

**Usage Pattern**:
```
Prompt 1: "Set up Node.js project with TypeScript for Stripe webhook processing"
Prompt 2-3: Package configuration and Stripe SDK setup
Prompts 4-8: Environment variables setup, webhook endpoint structure
Prompts 9-12: Basic server setup and validation
```

**Files Created**:
- `package.json` - Stripe, Express, TypeScript dependencies
- `server.ts` - Basic Express server with webhook endpoint
- `.env.example` - Environment variables template
- `types.ts` - TypeScript interfaces for customer data

#### Step 4-6: Webhook Architecture Design
**Claude Code Prompts**: 12-18 prompts  
**Opus 4 Time**: ~2-3 hours  
**Web Messages**: 15 messages (webhook security research)

**Usage Pattern**:
```
Prompts 1-6: Stripe webhook signature verification and security
Prompts 7-12: Customer data extraction logic and state normalization  
Prompts 13-18: Campaign name generation algorithm and edge cases
```

**Files Created**:
- `webhook-handler.ts` - Stripe event processing
- `data-extractor.ts` - Customer data extraction utilities
- `campaign-generator.ts` - State+date campaign naming logic
- `state-normalizer.ts` - US state validation and normalization

#### Step 7-10: MCP Server Integration
**Claude Code Prompts**: 10-15 prompts  
**Sonnet 4 Time**: ~2.5 hours  
**Web Messages**: 10 messages (MCP documentation)

**Usage Pattern**:
```
Prompts 1-5: Instantly MCP server configuration
Prompts 6-10: Campaign creation and lead management functions
Prompts 11-15: Error handling and retry logic
```

**Files Created**:
- `instantly-client.ts` - MCP server interface
- `campaign-manager.ts` - Campaign creation utilities
- `lead-creator.ts` - Lead management functions
- `config/claude-desktop.json` - MCP configuration

---

## Phase 2: Integration Implementation (Steps 11-25)
**Resource Estimate**: 50-70 Claude Code prompts + 60 web messages  
**Sonnet 4 Time**: ~10-12 hours  
**Goal**: Complete Stripe to Instantly data flow

### Resource Breakdown by Component

#### Step 11-15: Webhook Processing Pipeline
**Claude Code Prompts**: 15-20 prompts  
**Sonnet 4 Time**: ~3 hours  
**Web Messages**: 15 messages (event processing research)

**Files Created**:
- `webhook-processor.ts` - Main webhook processing logic
- `event-validator.ts` - Stripe event validation
- `error-handler.ts` - Webhook error handling and retries

#### Step 16-20: Campaign Management System  
**Claude Code Prompts**: 20-25 prompts  
**Sonnet 4 Time**: ~4 hours  
**Web Messages**: 20 messages (campaign optimization research)

**Files Created**:
- `campaign-orchestrator.ts` - Campaign creation orchestration
- `lead-processor.ts` - Lead creation and data mapping
- `duplicate-handler.ts` - Duplicate customer detection

#### Step 21-25: Integration Testing
**Claude Code Prompts**: 15-20 prompts  
**Sonnet 4 Time**: ~3 hours  
**Web Messages**: 25 messages (testing strategy research)

**Files Created**:
- `webhook.test.ts` - Webhook processing tests
- `campaign.test.ts` - Campaign creation tests
- `integration.test.ts` - End-to-end integration tests

---

## Phase 3: Email Automation System (Steps 26-40)  
**Resource Estimate**: 45-60 Claude Code prompts + 50 web messages  
**Sonnet 4 Time**: ~8-10 hours  
**Goal**: Annual reminder and cross-sell email sequences

### Resource Breakdown by Component

#### Step 26-30: Email Sequence Design
**Claude Code Prompts**: 15-20 prompts  
**Sonnet 4 Time**: ~3 hours  
**Web Messages**: 20 messages (email marketing research)

**Files Created**:
- `email-sequences.ts` - Sequence definitions and timing
- `template-generator.ts` - Dynamic email content generation
- `personalization.ts` - Customer data personalization

#### Step 31-35: Cross-Sell Automation  
**Claude Code Prompts**: 20-25 prompts  
**Sonnet 4 Time**: ~4 hours  
**Web Messages**: 20 messages (cross-sell strategy research)

**Files Created**:
- `cross-sell-engine.ts` - Product recommendation logic
- `upsell-sequences.ts` - Product-specific email sequences
- `revenue-tracker.ts` - Cross-sell performance tracking

#### Step 36-40: Automation Triggers
**Claude Code Prompts**: 15-20 prompts  
**Sonnet 4 Time**: ~3 hours  
**Web Messages**: 15 messages (automation timing research)

**Files Created**:
- `trigger-manager.ts` - Sequence triggering logic
- `date-calculator.ts` - Renewal date calculations
- `scheduler.ts` - Email timing and scheduling

---

## Phase 4: Production & Monitoring (Steps 41-50)
**Resource Estimate**: 35-45 Claude Code prompts + 40 web messages  
**Sonnet 4 Time**: ~6-8 hours  
**Goal**: Production deployment and performance monitoring

### Resource Breakdown by Component

#### Step 41-45: Production Setup
**Claude Code Prompts**: 15-20 prompts  
**Sonnet 4 Time**: ~3 hours  
**Web Messages**: 20 messages (deployment research)

**Files Created**:
- `production-config.ts` - Production environment configuration
- `health-check.ts` - System health monitoring
- `rate-limiter.ts` - API rate limiting and optimization

#### Step 46-50: Monitoring & Analytics
**Claude Code Prompts**: 20-25 prompts  
**Sonnet 4 Time**: ~4 hours  
**Web Messages**: 20 messages (monitoring best practices)

**Files Created**:
- `performance-monitor.ts` - System performance tracking
- `business-analytics.ts` - Campaign and revenue analytics
- `alert-system.ts` - Error alerting and notifications
- `dashboard.ts` - Performance dashboard and reporting

---

## 📊 Total Resource Requirements

### Complete Integration Summary
| Resource Type | Total Required | $200 Max Plan Capacity | Utilization |
|---------------|----------------|-------------------------|-------------|
| **Claude Code Prompts** | 160-215 prompts | 700-2,800/week | 5.5-31% |
| **Sonnet 4 Hours** | 30-38 hours | 280 hours/week | 10.5-13.5% |
| **Opus 4 Hours** | 2-3 hours | 30 hours/week | 6.5-10% |
| **Web Messages** | 185 messages | 3,150/week | 6% |
| **Implementation Timeframe** | 2-3 days | 1 week capacity | Very Comfortable |

### Resource Distribution by Phase
| Phase | Sonnet 4 Hours | Opus 4 Hours | % of Weekly Limit | Prompts | Web Messages |
|-------|----------------|--------------|-------------------|---------|--------------|
| Phase 1 | 4-5 hours | 2-3 hours | 2-3% | 30-40 | 35 |
| Phase 2 | 10-12 hours | 0 hours | 3.5-4.5% | 50-70 | 60 |
| Phase 3 | 8-10 hours | 0 hours | 3-3.5% | 45-60 | 50 |
| Phase 4 | 6-8 hours | 0 hours | 2-3% | 35-45 | 40 |

---

## 🚨 Resource Management Strategies

### Session Optimization for Customer Retention Focus
1. **Batch Related Integrations**: Group Stripe and Instantly work within sessions
2. **Business Logic Complexity**: Use Opus 4 for complex campaign naming and state logic
3. **Context Preservation**: Maintain customer data flow context for efficiency
4. **Validation Breaks**: Use web interface for business requirement research

### Rate Limit Management for Integration Project
1. **Weekly Planning**: Distribute webhook and email work across multiple days
2. **Model Selection**: Sonnet 4 for implementation, Opus 4 for business logic
3. **Auto-switching**: Allow fallback from Opus to Sonnet for routine tasks
4. **Buffer Management**: Keep 25% capacity buffer for testing and troubleshooting

### Recovery Strategies
1. **Daily Limits Hit**: Switch to documentation and business requirement analysis
2. **Weekly Limits Hit**: Focus on manual testing and campaign optimization
3. **Complex Issues**: Break down webhook processing into smaller focused prompts
4. **Integration Testing**: Use web interface for end-to-end validation

---

## 🔄 Phase Execution Strategy

### Day 1: Foundation (Phase 1 Complete)
- **Morning Session (5h)**: Architecture + webhook design (6-8 hours total)
- **Afternoon Session (5h)**: MCP integration and testing
- **Usage**: ~8 hours Sonnet 4, 3 hours Opus 4, 40 prompts, 35 web messages

### Day 2: Integration (Phase 2 Complete)  
- **Morning Session (5h)**: Webhook processing pipeline (5-6 hours Sonnet 4)
- **Afternoon Session (5h)**: Campaign management system (5-6 hours Sonnet 4)
- **Usage**: ~12 hours Sonnet 4, 65 prompts, 60 web messages

### Day 3: Automation + Production (Phases 3-4)
- **Morning Session (5h)**: Email sequences and cross-sell (8-10 hours Sonnet 4)
- **Afternoon Session (5h)**: Production setup and monitoring (6-8 hours Sonnet 4)
- **Usage**: ~16 hours Sonnet 4, 80 prompts, 90 web messages

### Weekly Resource Consumption
- **Total Sonnet 4**: 32-40 hours (11.5-14.5% of weekly limit)
- **Total Opus 4**: 2-3 hours (6.5-10% of weekly limit)
- **Total Prompts**: 185 prompts (26.5% of conservative session estimate)
- **Total Web Messages**: 185 messages (6% of weekly limit)
- **Buffer Remaining**: 240-248 Sonnet 4 hours (85.5-88.5% for other projects)

---

## ✅ Success Validation Checkpoints

### After Each Phase - Integration Check
```bash
# Test webhook processing
curl -X POST http://localhost:3000/stripe-webhook \
  -H "Content-Type: application/json" \
  -d @test-data/purchase-event.json

# Verify campaign creation in Instantly
# Check Claude Desktop MCP connection status

# Validate email sequence triggering
npm run test:sequences
```

### Resource Threshold Alerts
- **75% Sonnet 4 Weekly**: Focus on essential integration features only
- **90% Session Prompts**: Switch to web interface for business analysis
- **80% Web Messages**: Use offline documentation and testing
- **95% Any Limit**: Stop development, focus on manual validation

### Integration Quality Gates
1. **Phase 1**: Webhook processes Stripe events and generates campaign names
2. **Phase 2**: Customers are created as leads in correct Instantly campaigns  
3. **Phase 3**: Email sequences are triggered with proper timing and content
4. **Phase 4**: System is monitored and production-ready with alerting

---

## 🤖 Model Usage Strategy by Phase

### Phase 1: Architecture Foundation - Strategic Model Switching
```markdown
Steps 1-3: Project Setup (@model:sonnet-4)
- Node.js server setup and basic webhook endpoint
- Package management and TypeScript configuration
- Environment setup and development tooling

Steps 4-6: Webhook Architecture Design (@model:opus-4) 
- Complex state normalization and validation logic
- Campaign name generation algorithm with edge cases
- Multi-event webhook processing architecture
- Business logic for customer data extraction

Steps 7-10: MCP Integration (@model:sonnet-4)
- Instantly MCP server configuration and testing
- Campaign creation functions and error handling
- Integration testing and validation
```

### Phase 2: Integration Implementation (@model:sonnet-4 throughout)
**Rationale**: Standard API integration patterns
- Stripe webhook event processing and validation
- Instantly API integration and data mapping
- Error handling and retry logic implementation

### Phase 3: Email Automation (@model:sonnet-4 throughout)  
**Rationale**: Email sequence and automation patterns
- Annual reminder sequence design and timing
- Cross-sell automation logic and triggers
- Email template generation and personalization

### Phase 4: Production & Monitoring (@model:sonnet-4 throughout)
**Rationale**: Standard deployment and monitoring patterns
- Production configuration and environment setup
- Performance monitoring and business analytics
- Health checks and alerting systems

---

## 📊 Resource Efficiency Analysis

### Before Model Optimization:
- **Opus 4 Usage**: 6-8 hours for entire project (20-27% of weekly limit)
- **Risk**: Depleting premium model capacity on routine integration tasks
- **Inefficiency**: Using expensive model for standard API integrations

### After Model Optimization:
- **Opus 4 Usage**: 2-3 hours for Phase 1 only (6.5-10% of weekly limit) 
- **Efficiency Gain**: 60-70% reduction in Opus 4 usage
- **Quality Maintained**: Complex business logic still gets premium intelligence
- **Capacity Preserved**: 27-68 Opus 4 hours remaining for other projects

### Expected Weekly Resource Usage:
```
Brian Integration Consumption:
├── Opus 4: 2-3 hours (6.5-10% of weekly 30-70h limit)
├── Sonnet 4: 30-37 hours (10.5-13% of weekly 280-560h limit)  
├── Remaining Opus 4: 27-68 hours for other projects
└── Remaining Sonnet 4: 243-530 hours for other projects
```

### ROI Analysis:
- **Same Quality**: Complex webhook logic receives Opus 4 intelligence
- **Better Efficiency**: 60-70% reduction in premium model usage
- **More Capacity**: Massive resources preserved for other integrations
- **Optimal Value**: Maximum benefit from $200 Max Plan investment

---

*Resource planning optimized for customer retention automation system within $200 Max Plan limits. Designed for efficient integration development while maintaining enterprise-quality webhook processing and email automation.*