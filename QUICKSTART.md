# Quick Start Guide: Zapier + GPT-5 Pro Lead Processing

## Overview

This repository contains complete documentation for implementing an automated lead scoring and cleaning workflow using Zapier Platform and GPT-5 Pro AI. The system processes pre-foreclosure and distressed property leads, calculating sophisticated scoring metrics and routing leads to optimal activation channels.

## What This Workflow Does

### Input
- Upload CSV files containing raw lead data through a Zapier interface
- Supports multiple list types: Pre-Foreclosure (PFC), Divorce, Probate, Tax Lien, Pre-Probate

### Processing
1. **Data Cleaning**: AI-powered cleaning of names, addresses, phone numbers, and emails
2. **Scoring**: Calculates four key indices:
   - **DI (Distress Index)**: Measures financial distress/motivation (0-1 scale)
   - **UI (Urgency Index)**: Measures time sensitivity (0-1 scale)
   - **OLS (Offer-Likelihood Score)**: Predicts probability of accepting an offer (0-1 scale)
   - **CI (Contact-ability Index)**: Measures ability to reach the lead (0-1 scale)
   - **OPS (Overall Profitability Score)**: Master score combining all indices (0-1 scale)
3. **Classification**: Assigns tiers (High/Medium/Low) for distress, urgency, and reachability
4. **Quality Assessment**: Flags data quality (Clean, DefaultsOnly, Incomplete, Invalid)

### Output
- **Channel Assignment**: Routes leads to optimal activation channels:
  - Call+SMS (high-touch)
  - SMS-Only (medium-touch)
  - Email+SMS
  - Email-Only
  - NurtureMail (low-touch nurture campaigns)
- **Offer Strategy**: Assigns Cash-Only, Novation/Wholetail, or Cash-or-Novation
- **Playbook**: Assigns engagement playbook (e.g., PB-PFC-Standard, PB-Divorce-Nurture)
- **Activation**: Automatically sends leads to dialers, email platforms, or review queues

**Note**: This workflow **excludes all direct mail strategies** as specified in the requirements.

## Key Features

✅ **Fully Automated**: No manual data entry after CSV upload  
✅ **AI-Powered**: GPT-5 Pro handles complex data cleaning and decision logic  
✅ **Scalable**: Process thousands of leads per day  
✅ **Compliant**: Built-in DNC awareness and compliance checks  
✅ **Flexible**: Easy to adjust scoring weights and routing rules  
✅ **Auditable**: Complete logging of all AI decisions  

## Documentation Files

### 1. `zapier-workflow-guide.md` (Main Technical Documentation)
**Size**: 1,416 lines | 41 KB

**Contents**:
- Complete system architecture and data flow diagrams
- Detailed Zapier Tables schema for input/output tables
- Step-by-step workflow configuration (18 actions)
- Mathematical formulas for all scoring metrics with examples
- GPT-5 Pro prompt templates for each processing step
- Channel routing decision trees
- Quality assurance and monitoring guidelines
- Troubleshooting and best practices
- Implementation checklist (5-week plan)

**Use this for**: Understanding the complete system, technical implementation, and customization

### 2. `zapier-ai-prompts.md` (Configuration Prompts)
**Size**: 1,206 lines | 36 KB

**Contents**:
- Ready-to-use prompts for Zapier AI
- Action-by-action configuration instructions
- Copy-paste GPT-5 Pro prompts for all 14 AI processing steps
- Complete setup prompt (single paste option)
- Test data upload templates
- Modification prompts (DNC integration, email verification)

**Use this for**: Actually building the workflow in Zapier

### 3. `combined_clean_master.csv` (Sample Data)
**Size**: 1,315 leads

**Contents**:
- Example of processed lead data showing expected output format
- Includes all scoring fields (OPS, DI, UI, OLS, CI)
- Shows channel plans, offer strategies, and playbooks
- Demonstrates tier assignments and quality flags

**Use this for**: Understanding the data structure and expected outcomes

## Quick Implementation Steps

### Phase 1: Setup (1-2 hours)
1. Create Zapier account with Tables and OpenAI integration access
2. Obtain OpenAI API key with GPT-5 Pro access
3. Review `zapier-workflow-guide.md` Section 2 (Zapier Platform Configuration)
4. Create `lead_intake` and `lead_processed` tables using provided schemas

### Phase 2: Build Workflow (2-4 hours)
1. Open `zapier-ai-prompts.md`
2. Copy the "Complete Setup Prompt" (near the end of the document)
3. Paste into Zapier AI chat or workflow builder
4. Alternatively, build action-by-action using individual prompts
5. Configure GPT-5 Pro settings (temperature: 0.1, response format: json_object)

### Phase 3: Test (30 minutes)
1. Use the "Test Data Upload Prompt" from `zapier-ai-prompts.md`
2. Process 10-20 sample leads
3. Verify scores are in expected ranges (0-1)
4. Check channel assignments make logical sense
5. Review JSON outputs from GPT-5 Pro actions

### Phase 4: Connect Integrations (1-2 hours)
1. Configure webhooks to your dialer system
2. Connect email marketing platform (Mailchimp, ActiveCampaign, etc.)
3. Set up Slack notifications (optional)
4. Configure error handling and retry logic

### Phase 5: Production Launch (ongoing)
1. Process first full batch (100-500 leads)
2. Monitor for 48 hours
3. Gather feedback from acquisition team
4. Iterate on prompts based on results

## Scoring Formulas (Quick Reference)

### Distress Index (DI)
```
DI = (Equity_Normalized × 0.35) + (Event_Urgency × 0.40) + (Condition_Penalty × 0.25)
```
- High DI (>0.70): Highly distressed, motivated to sell
- Medium DI (0.40-0.69): Moderate distress
- Low DI (<0.40): Limited distress indicators

### Urgency Index (UI)
```
UI = Time_Decay × Event_Weight
```
- High UI (>0.70): Time-sensitive opportunity (0-30 days)
- Medium UI (0.40-0.69): Moderate urgency (31-90 days)
- Low UI (<0.40): Long-term opportunity (90+ days)

### Overall Profitability Score (OPS)
```
OPS = (DI × 0.30) + (UI × 0.25) + (OLS × 0.25) + (CI × 0.20)
```
- Tier 1 (0.80-1.00): Hot leads - immediate action
- Tier 2 (0.60-0.79): Warm leads - priority follow-up
- Tier 3 (0.40-0.59): Nurture - longer-term cultivation
- Tier 4 (0.00-0.39): Low priority - minimal resources

## Channel Routing Logic (Quick Reference)

```
High OPS (≥0.60) + High CI (≥0.70) + Has Cell → Call+SMS
High CI + Has Cell (but lower OPS) → SMS-Only
Medium CI (≥0.50) + Has Email → Email+SMS or Email-Only
Has Email only (low CI) → NurtureMail
Invalid/Incomplete → HOLD or REJECT
```

**Important**: All direct mail strategies are excluded from this workflow.

## Support and Customization

### Adjusting Scoring Weights
To change how scores are calculated, edit the formulas in the GPT-5 Pro prompts. For example, to increase the weight of urgency in the OPS calculation:

```
Change: OPS = (DI × 0.30) + (UI × 0.25) + (OLS × 0.25) + (CI × 0.20)
To:     OPS = (DI × 0.25) + (UI × 0.35) + (OLS × 0.20) + (CI × 0.20)
```

Ensure all weights sum to 1.0.

### Adding New Event Types
Edit Action 6 (DI calculation) and Action 7 (UI calculation) prompts to include new event types and their urgency scores.

### Modifying Channel Plans
Edit Action 12 (Channel Assignment) prompt to add new channels or adjust routing rules.

### Integrating DNC Services
Use the "Add DNC Integration" prompt from `zapier-ai-prompts.md` to integrate with Do Not Call registries.

## Cost Estimates

### Zapier Costs
- Professional Plan (recommended): $49/month for 2,000 tasks
- Each lead processes through ~20 actions = 20 tasks
- **Capacity**: ~100 leads/month on Professional plan
- For higher volume, consider Team ($299/month, 50,000 tasks = ~2,500 leads)

### OpenAI API Costs
- GPT-5 Pro pricing: ~$0.01-0.03 per lead (14 API calls × ~500 tokens avg)
- 1,000 leads/month = ~$10-30 in OpenAI costs
- Use GPT-4 Turbo for simple cleaning tasks to reduce costs

### Total Estimated Cost
- 100 leads/month: ~$50-60/month
- 1,000 leads/month: ~$60-90/month
- 5,000 leads/month: ~$350-450/month

## Data Privacy and Compliance

⚠️ **Important Compliance Notes**:
1. Leads with phone numbers on Do Not Call (DNC) registries are automatically routed to email-only channels
2. All data processing happens within Zapier's secure environment
3. No lead data is stored permanently by GPT-5 Pro (processed in real-time)
4. Ensure your Zapier account has appropriate access controls
5. This workflow does not include direct mail to comply with user requirements
6. Consult with legal counsel regarding data retention and privacy regulations (TCPA, GDPR, CCPA)

## Next Steps

1. **Read First**: `zapier-workflow-guide.md` Sections 1-2 for architecture understanding
2. **Build**: Use prompts from `zapier-ai-prompts.md` to construct the workflow
3. **Test**: Process sample data from `combined_clean_master.csv`
4. **Launch**: Deploy to production with monitoring
5. **Optimize**: Iterate on prompts based on feedback

## Questions or Issues?

Refer to:
- **Technical Questions**: `zapier-workflow-guide.md` Section 11 (Troubleshooting)
- **Configuration Issues**: `zapier-ai-prompts.md` modification prompts
- **Scoring Logic**: `zapier-workflow-guide.md` Section 5 (Scoring Logic & Equations)
- **Data Quality**: `zapier-workflow-guide.md` Section 6 (Data Cleaning & Validation)

## Version Information

- **Documentation Version**: 1.0
- **Created**: November 2025
- **Last Updated**: November 6, 2025
- **Zapier Platform**: Tables + Workflows
- **AI Model**: GPT-5 Pro (or GPT-4 Turbo fallback)
- **Data Format**: CSV input, Zapier Tables storage

---

**Ready to get started?** Open `zapier-ai-prompts.md` and begin with the "Initial Setup Prompt"!
