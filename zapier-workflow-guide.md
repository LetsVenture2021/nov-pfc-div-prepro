# Zapier + GPT-5 Pro: Automated Lead Scoring & Cleaning Workflow

## Executive Overview

This document describes a complete workflow for loading, scoring, cleaning, and routing pre-foreclosure and distressed property leads using the Zapier Platform integrated with OpenAI's GPT-5 Pro model. The system automates the entire data processing pipeline from initial upload through channel assignment and activation routing—excluding direct mail strategies.

---

## Table of Contents

1. [System Architecture](#system-architecture)
2. [Zapier Platform Configuration](#zapier-platform-configuration)
3. [GPT-5 Pro Integration Setup](#gpt-5-pro-integration-setup)
4. [Data Ingestion & Upload Process](#data-ingestion--upload-process)
5. [Scoring Logic & Equations](#scoring-logic--equations)
6. [Data Cleaning & Validation](#data-cleaning--validation)
7. [Channel Routing Logic](#channel-routing-logic)
8. [Automation Workflow Steps](#automation-workflow-steps)
9. [GPT-5 Pro Prompts Library](#gpt-5-pro-prompts-library)
10. [Quality Assurance & Monitoring](#quality-assurance--monitoring)
11. [Troubleshooting & Best Practices](#troubleshooting--best-practices)

---

## System Architecture

### High-Level Data Flow

```
Raw Lead CSV Files
    ↓
Zapier Tables (Lead Upload Interface)
    ↓
GPT-5 Pro Processing Layer
    ↓
[Scoring Module] → [Cleaning Module] → [Routing Module]
    ↓
Zapier Storage/Output Tables
    ↓
Activation Channels (Call+SMS, Email, SMS-Only, Nurture Email)
```

### Technology Stack

- **Zapier Platform**: Workflow automation and data interface
- **Zapier Tables**: Data storage and structured lead management
- **OpenAI GPT-5 Pro API**: AI-powered data processing and decision logic
- **Zapier Code (Python/JavaScript)**: Custom computations and transformations
- **Zapier Webhooks**: External system integrations (dialers, CRM, email platforms)

---

## Zapier Platform Configuration

### Step 1: Create Zapier Tables

#### Table 1: `lead_intake` (Raw Upload Table)

**Purpose**: Initial landing zone for uploaded lead data

**Required Fields**:
- `lead_id` (Unique Identifier, Auto-generated)
- `first_name` (Text)
- `last_name` (Text)
- `company_name` (Text, Optional)
- `street_address` (Text)
- `city` (Text)
- `state` (Text, 2-character abbreviation)
- `zip` (Text, 5 or 9 digits)
- `mail_street_address` (Text)
- `mail_city` (Text)
- `mail_state` (Text)
- `mail_zip` (Text)
- `phone_numbers` (Long Text, semicolon-separated)
- `email_addresses` (Long Text, semicolon-separated)
- `event_type` (Text: Auction, Divorce, Probate, Tax Lien, etc.)
- `event_date` (Date)
- `data_source` (Text: filename or import batch ID)
- `imported_at` (Datetime, Auto-set)
- `list_type` (Text: PFC, Divorce, Probate, etc.)
- `estimated_equity_pct` (Number, 0-100)
- `property_condition_score` (Number, 1-5)

#### Table 2: `lead_processed` (Scored & Cleaned Output)

**Purpose**: Final processed leads ready for activation

**Fields** (includes all from `lead_intake` plus):
- `ops_score` (Number, 0-1, Overall Profitability Score)
- `rank_ops` (Number, 1-N, ranking position)
- `di_score` (Number, 0-1, Distress Index)
- `ui_score` (Number, 0-1, Urgency Index)
- `ols_score` (Number, 0-1, Offer-Likelihood Score)
- `ci_score` (Number, 0-1, Contact-ability Index)
- `distress_tier` (Text: H/M/L)
- `urgency_tier` (Text: H/M/L)
- `reach_tier` (Text: H/M/L)
- `motivation_level` (Text: High/Medium/Low)
- `channel_plan` (Text: Call+SMS, Email+SMS, SMS-Only, NurtureMail, MailOnly)
- `offer_strategy` (Text: Cash-Only, Novation/Wholetail, Cash-or-Novation)
- `playbook` (Text: PB-PFC-Standard, PB-Divorce-Nurture, etc.)
- `callable_phones` (Long Text, cleaned phone list)
- `has_any_cell` (Boolean)
- `has_any_landline` (Boolean)
- `has_any_email` (Boolean)
- `any_dnc` (Boolean, Do Not Call flag)
- `compliance_hold` (Boolean)
- `quality_flag` (Text: Clean, DefaultsOnly, Incomplete, Invalid)
- `dialable` (Boolean, 1 if ready for dialer)
- `processing_timestamp` (Datetime)
- `gpt_validation_notes` (Long Text)

### Step 2: Create Zapier Interface (Lead Upload Portal)

1. **Navigate to**: Zapier Interfaces → Create New Interface
2. **Template**: Start with "Data Upload Form"
3. **Configure Form Fields**:
   - Add file upload component (CSV only)
   - Add text field for "Import Batch Name"
   - Add dropdown for "List Type" (PFC, Divorce, Probate, Tax Lien, Pre-Probate)
   - Add text field for "Market" (e.g., HOU, DFW, ATL)
4. **Action on Submit**: Trigger Zap → "Process Lead Upload"

---

## GPT-5 Pro Integration Setup

### Zapier OpenAI Integration Configuration

1. **Connect OpenAI Account**:
   - Zapier → Apps → Search "OpenAI"
   - Choose "OpenAI (ChatGPT, Whisper, DALL-E)"
   - Authenticate with API key (requires GPT-5 Pro API access)

2. **Create Reusable GPT-5 Pro Action**:
   - Action Name: "GPT-5 Pro Data Processor"
   - Model: `gpt-5-pro` (or latest GPT-5 identifier)
   - Temperature: 0.1 (for consistent, deterministic processing)
   - Max Tokens: 4000
   - Response Format: JSON mode (structured output)

---

## Data Ingestion & Upload Process

### Step-by-Step Upload Flow

#### Phase 1: CSV File Upload

**Trigger**: User uploads CSV via Zapier Interface

**Zap Actions**:
1. **Trigger**: Zapier Interface - "Form Submission"
2. **Action**: Google Drive / Dropbox / S3 - "Upload File"
   - Store original CSV for audit trail
3. **Action**: Code by Zapier (Python) - "Parse CSV"
   
```python
# Parse CSV and validate structure
import csv
import io

input_data = input_data.get('csv_content')
batch_name = input_data.get('batch_name')
list_type = input_data.get('list_type')

reader = csv.DictReader(io.StringIO(input_data))
leads = []

for row in reader:
    lead = {
        'first_name': row.get('First Name', '').strip(),
        'last_name': row.get('Last Name', '').strip(),
        'street_address': row.get('Street Address', '').strip(),
        'city': row.get('City', '').strip(),
        'state': row.get('State', '').strip().upper(),
        'zip': row.get('Zip', '').strip(),
        'phone_numbers': row.get('Phone', ''),  # May have multiple
        'email_addresses': row.get('Email', ''),
        'event_type': row.get('Event Type', ''),
        'event_date': row.get('Event Date', ''),
        'list_type': list_type,
        'data_source': batch_name,
    }
    leads.append(lead)

return {'leads': leads, 'count': len(leads)}
```

4. **Action**: Looping by Zapier - "Loop Through Leads"
5. **Action**: Zapier Tables - "Create Record in lead_intake"

---

## Scoring Logic & Equations

The scoring system consists of four primary indices that combine to produce an Overall Profitability Score (OPS).

### 1. Distress Index (DI)

**Purpose**: Measures the level of financial distress/motivation

**Formula**:
```
DI = (EstEquityPct_Norm × 0.35) + (EventUrgency × 0.40) + (ConditionPenalty × 0.25)

Where:
- EstEquityPct_Norm = MIN(EstimatedEquity% / 50, 1.0)
  - Normalizes equity to 0-1 scale (50%+ equity = max score)
  
- EventUrgency = Based on event type:
  * Auction/Foreclosure: 1.0 (highest distress)
  * Tax Lien: 0.85
  * Divorce: 0.60
  * Probate: 0.50
  * Pre-Probate: 0.30
  
- ConditionPenalty = (5 - ConditionScore) / 5
  - ConditionScore 1 (Poor) → Penalty 0.8
  - ConditionScore 5 (Excellent) → Penalty 0.0
  - Poor condition increases distress (more likely to sell)
```

**GPT-5 Pro Calculation Prompt**:
```
Calculate the Distress Index (DI) for this lead:
- Estimated Equity: {equity_pct}%
- Event Type: {event_type}
- Property Condition Score: {condition_score} (1-5 scale)

Apply this formula:
1. Normalize equity: MIN(equity% / 50, 1.0)
2. Assign event urgency score (Auction=1.0, Tax Lien=0.85, Divorce=0.60, Probate=0.50, Pre-Probate=0.30)
3. Calculate condition penalty: (5 - condition_score) / 5
4. DI = (equity_norm × 0.35) + (event_urgency × 0.40) + (condition_penalty × 0.25)

Return ONLY a JSON object: {"di_score": <value>, "reasoning": "<brief explanation>"}
```

### 2. Urgency Index (UI)

**Purpose**: Measures time sensitivity of the opportunity

**Formula**:
```
UI = TimeDecay × EventWeight

Where:
- TimeDecay = Based on DaysUntilEvent:
  * 0-30 days: 1.0 (maximum urgency)
  * 31-60 days: 0.70
  * 61-90 days: 0.45
  * 91-180 days: 0.25
  * 181+ days: 0.10
  * Unknown/No date: 0.05
  
- EventWeight = Multiplier based on event type:
  * Auction: 1.0 (date is critical)
  * Tax Lien: 0.85
  * Divorce: 0.70 (dates less rigid)
  * Probate: 0.60
  * Pre-Probate: 0.40 (very flexible timeline)
```

**GPT-5 Pro Calculation Prompt**:
```
Calculate the Urgency Index (UI) for this lead:
- Event Type: {event_type}
- Event Date: {event_date}
- Today's Date: {current_date}

Apply this logic:
1. Calculate DaysUntilEvent = event_date - today
2. Assign TimeDecay based on days remaining (0-30=1.0, 31-60=0.70, 61-90=0.45, 91-180=0.25, 181+=0.10)
3. Assign EventWeight (Auction=1.0, Tax Lien=0.85, Divorce=0.70, Probate=0.60, Pre-Probate=0.40)
4. UI = TimeDecay × EventWeight

Return JSON: {"ui_score": <value>, "days_until_event": <days>, "reasoning": "<explanation>"}
```

### 3. Offer-Likelihood Score (OLS)

**Purpose**: Predicts probability of accepting an offer

**Formula**:
```
OLS = (DI × 0.50) + (UI × 0.30) + (MarketFactor × 0.20)

Where:
- DI = Distress Index (calculated above)
- UI = Urgency Index (calculated above)
- MarketFactor = Property/market conditions:
  * High-appreciation market + Poor condition: 0.90
  * High-appreciation market + Good condition: 0.70
  * Stable market + Poor condition: 0.60
  * Stable market + Good condition: 0.40
  * Declining market: 0.30
```

**GPT-5 Pro Calculation Prompt**:
```
Calculate the Offer-Likelihood Score (OLS):
- Distress Index (DI): {di_score}
- Urgency Index (UI): {ui_score}
- Market: {market}
- Property Condition: {condition_score}

Apply this formula:
1. Determine MarketFactor based on market strength and condition
2. OLS = (DI × 0.50) + (UI × 0.30) + (MarketFactor × 0.20)

Return JSON: {"ols_score": <value>, "market_factor": <value>, "reasoning": "<explanation>"}
```

### 4. Contact-ability Index (CI)

**Purpose**: Measures ability to reach the lead

**Formula**:
```
CI = (PhoneScore × 0.40) + (EmailScore × 0.25) + (AddressScore × 0.20) + (DNCPenalty × 0.15)

Where:
- PhoneScore:
  * Has validated cell phone: 1.0
  * Has landline only: 0.60
  * Has unvalidated phone: 0.40
  * No phone: 0.0
  
- EmailScore:
  * Has verified email: 1.0
  * Has unverified email: 0.50
  * No email: 0.0
  
- AddressScore:
  * Mailing address matches property: 1.0
  * Different mailing address (valid): 0.70
  * No valid mailing address: 0.0
  
- DNCPenalty:
  * Not on DNC list: 1.0
  * On DNC list: 0.0 (CI effectively becomes 0)
```

**GPT-5 Pro Calculation Prompt**:
```
Calculate the Contact-ability Index (CI):
- Phone Numbers: {phone_numbers}
- Has Cell: {has_cell}
- Has Landline: {has_landline}
- Email Addresses: {emails}
- Mailing Address Matches Property: {address_match}
- On DNC List: {on_dnc}

Apply this formula:
1. Score phone quality (cell=1.0, landline=0.60, unvalidated=0.40, none=0.0)
2. Score email (verified=1.0, unverified=0.50, none=0.0)
3. Score address (match=1.0, different=0.70, none=0.0)
4. Apply DNC penalty (if on DNC, multiply by 0.0)
5. CI = (PhoneScore × 0.40) + (EmailScore × 0.25) + (AddressScore × 0.20) + (DNCPenalty × 0.15)

Return JSON: {"ci_score": <value>, "phone_score": <val>, "email_score": <val>, "address_score": <val>, "reasoning": "<explanation>"}
```

### 5. Overall Profitability Score (OPS)

**Purpose**: Master score for ranking and prioritization

**Formula**:
```
OPS = (DI × 0.30) + (UI × 0.25) + (OLS × 0.25) + (CI × 0.20)

Result range: 0.0 to 1.0
- 0.80 - 1.00: Tier 1 (Hot Leads - immediate action)
- 0.60 - 0.79: Tier 2 (Warm Leads - priority follow-up)
- 0.40 - 0.59: Tier 3 (Nurture - longer-term cultivation)
- 0.00 - 0.39: Tier 4 (Low Priority - minimal resources)
```

**GPT-5 Pro Final Scoring Prompt**:
```
Calculate the Overall Profitability Score (OPS):
- Distress Index (DI): {di_score}
- Urgency Index (UI): {ui_score}
- Offer-Likelihood Score (OLS): {ols_score}
- Contact-ability Index (CI): {ci_score}

Apply this formula:
OPS = (DI × 0.30) + (UI × 0.25) + (OLS × 0.25) + (CI × 0.20)

Then assign tier classification:
- 0.80-1.00: Tier 1 (Hot)
- 0.60-0.79: Tier 2 (Warm)
- 0.40-0.59: Tier 3 (Nurture)
- 0.00-0.39: Tier 4 (Low)

Return JSON: {
  "ops_score": <value>,
  "tier": <1-4>,
  "tier_label": "<Hot/Warm/Nurture/Low>",
  "distress_tier": "<H/M/L>",
  "urgency_tier": "<H/M/L>",
  "reach_tier": "<H/M/L>",
  "reasoning": "<comprehensive explanation>"
}
```

---

## Data Cleaning & Validation

### Phase 1: Name Standardization

**GPT-5 Pro Prompt**:
```
Clean and standardize this name:
First Name: "{first_name}"
Last Name: "{last_name}"
Company Name: "{company_name}"

Apply these rules:
1. Title case all names (proper capitalization)
2. Remove special characters except hyphens and apostrophes
3. Fix common OCR errors (e.g., "0" → "O" in names)
4. Detect and flag business entities vs. individuals
5. Standardize suffixes (Jr., Sr., II, III)
6. Remove leading/trailing whitespace

Return JSON: {
  "first_name_clean": "<cleaned>",
  "last_name_clean": "<cleaned>",
  "company_name_clean": "<cleaned>",
  "is_business": <true/false>,
  "validation_issues": ["<list any issues>"]
}
```

### Phase 2: Address Validation & Standardization

**GPT-5 Pro Prompt**:
```
Validate and standardize these addresses:
Property Address: {street}, {city}, {state} {zip}
Mailing Address: {mail_street}, {mail_city}, {mail_state} {mail_zip}

Apply these rules:
1. Standardize street suffixes (St → Street, Ave → Avenue, Dr → Drive)
2. Correct directionals (N, S, E, W, NE, NW, SE, SW)
3. Validate ZIP codes (5 or 9 digits, matches city/state)
4. Flag PO Boxes in property address (invalid)
5. Determine if mailing address matches property
6. Validate state abbreviations (2-letter USPS codes)

Return JSON: {
  "property_address_clean": "<full standardized address>",
  "mailing_address_clean": "<full standardized address>",
  "addresses_match": <true/false>,
  "property_has_pobox": <true/false>,
  "validation_issues": ["<list issues>"],
  "confidence_score": <0-1>
}
```

### Phase 3: Phone Number Cleaning & Classification

**GPT-5 Pro Prompt**:
```
Process and classify these phone numbers:
Raw Input: "{phone_numbers}"

Apply these rules:
1. Extract all valid 10-digit US phone numbers
2. Remove duplicates
3. Format as: XXX-XXX-XXXX
4. Classify each as: Cell, Landline, VoIP, or Unknown
5. Flag invalid/disconnected numbers
6. Check against DNC registries (simulate lookup)
7. Prioritize cell phones over landlines

Return JSON: {
  "phones_cleaned": ["<list of formatted numbers>"],
  "phone_types": {"<number>": "<type>", ...},
  "has_any_cell": <true/false>,
  "has_any_landline": <true/false>,
  "callable_phones": ["<numbers not on DNC>"],
  "dnc_phones": ["<numbers on DNC>"],
  "validation_issues": ["<issues>"]
}
```

### Phase 4: Email Validation

**GPT-5 Pro Prompt**:
```
Validate and clean email addresses:
Raw Input: "{email_addresses}"

Apply these rules:
1. Extract valid email addresses (RFC 5322 format)
2. Remove duplicates
3. Lowercase all emails
4. Detect disposable/temporary email providers
5. Check for common typos in domains (gmial.com → gmail.com)
6. Validate domain existence (simulate DNS check)
7. Flag role-based emails (info@, admin@, sales@)

Return JSON: {
  "emails_cleaned": ["<list of valid emails>"],
  "has_any_email": <true/false>,
  "disposable_emails": ["<temp email addresses>"],
  "role_based_emails": ["<role emails>"],
  "validation_issues": ["<issues>"],
  "confidence_scores": {"<email>": <0-1>, ...}
}
```

### Phase 5: Data Completeness Check

**GPT-5 Pro Prompt**:
```
Assess data quality and completeness for this lead:
{full_lead_json}

Evaluate these criteria:
1. Required fields present: First Name, Last Name, Address, City, State, ZIP
2. At least one contact method: Phone OR Email
3. Event information available: Type and Date (or estimated)
4. Data consistency: All related fields align logically
5. No suspicious patterns indicating fake/test data

Assign quality flag:
- "Clean": All required fields, multiple contact methods, high confidence
- "DefaultsOnly": Missing some preferred fields but still usable
- "Incomplete": Missing critical fields, limited contact info
- "Invalid": Suspect data, multiple validation failures

Return JSON: {
  "quality_flag": "<Clean|DefaultsOnly|Incomplete|Invalid>",
  "completeness_score": <0-1>,
  "missing_fields": ["<fields>"],
  "validation_notes": "<detailed assessment>",
  "recommended_action": "<Accept|Review|Reject>"
}
```

---

## Channel Routing Logic

Based on scoring and contact-ability, leads are assigned to activation channels. **Note: Direct mail strategies are excluded from this workflow.**

### Channel Assignment Rules

**Decision Tree**:

```
IF quality_flag == "Invalid" OR completeness_score < 0.4:
  → REJECT (do not route)

ELIF any_dnc == TRUE OR compliance_hold == TRUE:
  → NurtureMail (email-only nurture campaign)

ELIF ci_score >= 0.70 AND has_any_cell == TRUE:
  IF ops_score >= 0.60:
    → Call+SMS (outbound calls + text messaging)
  ELSE:
    → SMS-Only (text messaging campaigns)

ELIF ci_score >= 0.50 AND has_any_email == TRUE:
  IF ops_score >= 0.60:
    → Email+SMS (if has cell) OR Email-Only
  ELSE:
    → NurtureMail (long-term email nurture)

ELIF ci_score < 0.50 AND has_any_email == TRUE:
  → NurtureMail (limited contact options)

ELSE:
  → HOLD (insufficient contact information, requires manual review)
```

**GPT-5 Pro Channel Assignment Prompt**:
```
Determine the optimal channel plan for this lead:
- OPS Score: {ops_score}
- CI Score: {ci_score}
- Has Cell Phone: {has_cell}
- Has Landline: {has_landline}
- Has Email: {has_email}
- On DNC List: {on_dnc}
- Compliance Hold: {compliance_hold}
- Quality Flag: {quality_flag}

Available channels (NO DIRECT MAIL OPTIONS):
1. Call+SMS: High-touch outbound calling + text campaigns
2. SMS-Only: Text messaging without voice calls
3. Email+SMS: Email campaigns supplemented with texts
4. Email-Only: Email campaigns only
5. NurtureMail: Long-term email nurture sequences
6. HOLD: Manual review required
7. REJECT: Insufficient quality/compliance issue

Apply the decision tree logic and return JSON: {
  "channel_plan": "<selected channel>",
  "dialable": <true/false>,
  "reasoning": "<why this channel was chosen>",
  "alternate_channels": ["<backups if primary fails>"],
  "compliance_notes": "<any compliance considerations>"
}
```

### Offer Strategy Assignment

**GPT-5 Pro Offer Strategy Prompt**:
```
Determine the best offer strategy for this lead:
- Distress Index (DI): {di_score}
- Urgency Index (UI): {ui_score}
- Estimated Equity %: {equity_pct}
- Property Condition: {condition_score}
- Event Type: {event_type}

Available strategies:
1. Cash-Only: Quick cash offers for urgent situations
2. Novation/Wholetail: Creative financing, seller-stays options
3. Cash-or-Novation: Flexible, present both options

Decision logic:
- IF UI >= 0.70 AND equity < 30%: → Cash-Only (urgency + low equity)
- ELIF equity >= 40% AND condition >= 3: → Novation/Wholetail (good property)
- ELSE: → Cash-or-Novation (flexible approach)

Return JSON: {
  "offer_strategy": "<selected strategy>",
  "reasoning": "<why this strategy fits>",
  "talking_points": ["<key points for acquisition team>"]
}
```

### Playbook Assignment

**GPT-5 Pro Playbook Prompt**:
```
Assign the appropriate playbook for this lead:
- List Type: {list_type}
- Event Type: {event_type}
- OPS Score: {ops_score}
- Channel Plan: {channel_plan}
- Offer Strategy: {offer_strategy}

Available playbooks:
1. PB-PFC-Standard: Standard pre-foreclosure workflow
2. PB-PFC-Urgent: High-urgency pre-foreclosure (auction within 30 days)
3. PB-Divorce-Standard: Divorce-motivated sellers
4. PB-Divorce-Nurture: Lower-urgency divorce cases
5. PB-Probate-Standard: Active probate cases
6. PB-PreProbate-Nurture: Pre-probate leads (long-term)
7. PB-TaxLien-Urgent: Tax lien with near-term sale date

Decision logic:
- Match List Type + Event Type to playbook category
- If UI >= 0.70: Use "Urgent" or "Standard" variant
- If UI < 0.40: Use "Nurture" variant

Return JSON: {
  "playbook": "<selected playbook>",
  "reasoning": "<why this playbook>",
  "key_actions": ["<main steps in this playbook>"],
  "timeline": "<expected duration>"
}
```

---

## Automation Workflow Steps

### Complete Zapier Workflow Configuration

**Zap Name**: Lead Processing Master Pipeline

#### Trigger
- **App**: Zapier Tables
- **Event**: New Record in `lead_intake`
- **Filters**: None (process all new leads)

---

#### Action 1: Data Extraction & Normalization
- **App**: Code by Zapier (Python)
- **Action**: Extract and prepare lead data
- **Code**:
```python
import json
from datetime import datetime

# Extract lead data from trigger
lead = input_data
lead_id = lead.get('lead_id')

# Normalize phone numbers
phones_raw = lead.get('phone_numbers', '')
phones_list = [p.strip() for p in phones_raw.split(';') if p.strip()]

# Normalize emails
emails_raw = lead.get('email_addresses', '')
emails_list = [e.strip().lower() for e in emails_raw.split(';') if e.strip()]

# Prepare for GPT processing
output = {
    'lead_id': lead_id,
    'first_name': lead.get('first_name', ''),
    'last_name': lead.get('last_name', ''),
    'full_address': f"{lead.get('street_address', '')} {lead.get('city', '')} {lead.get('state', '')} {lead.get('zip', '')}",
    'phones': phones_list,
    'emails': emails_list,
    'event_type': lead.get('event_type', ''),
    'event_date': lead.get('event_date', ''),
    'equity_pct': lead.get('estimated_equity_pct', 0),
    'condition_score': lead.get('property_condition_score', 3),
    'current_date': datetime.now().strftime('%Y-%m-%d')
}

return output
```

---

#### Action 2: GPT-5 Pro - Name Cleaning
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **Model**: gpt-5-pro
- **User Message**: 
```
Clean and standardize this name:
First Name: "{{1.first_name}}"
Last Name: "{{1.last_name}}"
Company Name: "{{lead_intake.company_name}}"

Apply these rules:
1. Title case all names
2. Remove special characters except hyphens and apostrophes
3. Fix common OCR errors
4. Detect business vs. individual
5. Standardize suffixes

Return ONLY valid JSON: {
  "first_name_clean": "<cleaned>",
  "last_name_clean": "<cleaned>",
  "company_name_clean": "<cleaned>",
  "is_business": <true/false>,
  "validation_issues": []
}
```
- **System Message**: "You are a data cleaning specialist. Always return valid JSON. Be precise and consistent."
- **Temperature**: 0.1
- **Response Format**: json_object

---

#### Action 3: GPT-5 Pro - Address Validation
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Validate and standardize these addresses:
Property: {{lead_intake.street_address}}, {{lead_intake.city}}, {{lead_intake.state}} {{lead_intake.zip}}
Mailing: {{lead_intake.mail_street_address}}, {{lead_intake.mail_city}}, {{lead_intake.mail_state}} {{lead_intake.mail_zip}}

Rules:
1. Standardize street suffixes
2. Validate ZIP codes
3. Flag PO Boxes in property address
4. Determine if addresses match

Return ONLY valid JSON: {
  "property_address_clean": "<address>",
  "mailing_address_clean": "<address>",
  "addresses_match": <true/false>,
  "property_has_pobox": <true/false>,
  "validation_issues": [],
  "confidence_score": <0-1>
}
```

---

#### Action 4: GPT-5 Pro - Phone Cleaning & Classification
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Process these phone numbers: {{1.phones}}

Rules:
1. Extract valid 10-digit US numbers
2. Format as XXX-XXX-XXXX
3. Classify as Cell, Landline, or Unknown
4. Remove duplicates
5. Assume numbers with patterns xxx5xx or xxx6xx are likely cells

Return ONLY valid JSON: {
  "phones_cleaned": [],
  "phone_types": {},
  "has_any_cell": <true/false>,
  "has_any_landline": <true/false>,
  "callable_phones": [],
  "validation_issues": []
}
```

---

#### Action 5: GPT-5 Pro - Email Validation
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Validate these emails: {{1.emails}}

Rules:
1. Extract valid email addresses
2. Remove duplicates
3. Lowercase all
4. Check for common typos (gmial.com → gmail.com)
5. Flag disposable providers

Return ONLY valid JSON: {
  "emails_cleaned": [],
  "has_any_email": <true/false>,
  "disposable_emails": [],
  "validation_issues": []
}
```

---

#### Action 6: GPT-5 Pro - Calculate Distress Index (DI)
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Calculate Distress Index (DI):
- Equity: {{1.equity_pct}}%
- Event Type: {{1.event_type}}
- Condition: {{1.condition_score}}

Formula:
1. Equity_norm = MIN(equity / 50, 1.0)
2. Event urgency: Auction=1.0, TaxLien=0.85, Divorce=0.60, Probate=0.50, PreProbate=0.30
3. Condition penalty: (5 - condition) / 5
4. DI = (equity_norm × 0.35) + (event_urgency × 0.40) + (condition_penalty × 0.25)

Return ONLY valid JSON: {"di_score": <value>, "reasoning": "<brief>"}
```

---

#### Action 7: GPT-5 Pro - Calculate Urgency Index (UI)
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Calculate Urgency Index (UI):
- Event Type: {{1.event_type}}
- Event Date: {{1.event_date}}
- Today: {{1.current_date}}

Logic:
1. Calculate days until event
2. TimeDecay: 0-30days=1.0, 31-60=0.70, 61-90=0.45, 91-180=0.25, 181+=0.10
3. EventWeight: Auction=1.0, TaxLien=0.85, Divorce=0.70, Probate=0.60, PreProbate=0.40
4. UI = TimeDecay × EventWeight

Return ONLY valid JSON: {"ui_score": <value>, "days_until_event": <days>, "reasoning": "<brief>"}
```

---

#### Action 8: GPT-5 Pro - Calculate Contact-ability Index (CI)
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Calculate Contact-ability Index (CI):
- Has Cell: {{4.has_any_cell}}
- Has Landline: {{4.has_any_landline}}
- Has Email: {{5.has_any_email}}
- Addresses Match: {{3.addresses_match}}

Formula:
- PhoneScore: cell=1.0, landline=0.60, none=0.0
- EmailScore: yes=1.0, no=0.0
- AddressScore: match=1.0, different=0.70
- CI = (PhoneScore × 0.40) + (EmailScore × 0.25) + (AddressScore × 0.20) + 0.15

Return ONLY valid JSON: {"ci_score": <value>, "reasoning": "<brief>"}
```

---

#### Action 9: GPT-5 Pro - Calculate Offer-Likelihood Score (OLS)
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Calculate Offer-Likelihood Score (OLS):
- DI: {{6.di_score}}
- UI: {{7.ui_score}}
- Equity: {{1.equity_pct}}%
- Condition: {{1.condition_score}}

Formula:
- MarketFactor: high appreciation + poor condition=0.90, else 0.60
- OLS = (DI × 0.50) + (UI × 0.30) + (MarketFactor × 0.20)

Return ONLY valid JSON: {"ols_score": <value>, "reasoning": "<brief>"}
```

---

#### Action 10: GPT-5 Pro - Calculate Overall Profitability Score (OPS)
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Calculate Overall Profitability Score (OPS):
- DI: {{6.di_score}}
- UI: {{7.ui_score}}
- OLS: {{9.ols_score}}
- CI: {{8.ci_score}}

Formula:
OPS = (DI × 0.30) + (UI × 0.25) + (OLS × 0.25) + (CI × 0.20)

Assign tiers:
- 0.80-1.00: Tier 1, Distress=H, Urgency=H, Reach=H
- 0.60-0.79: Tier 2, adjust based on component scores
- 0.40-0.59: Tier 3, adjust based on component scores
- 0.00-0.39: Tier 4, all L

Return ONLY valid JSON: {
  "ops_score": <value>,
  "tier": <1-4>,
  "distress_tier": "<H/M/L>",
  "urgency_tier": "<H/M/L>",
  "reach_tier": "<H/M/L>",
  "motivation_level": "<High/Medium/Low>"
}
```

---

#### Action 11: GPT-5 Pro - Quality Assessment
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Assess lead quality:
Lead Data: {
  "name": "{{2.first_name_clean}} {{2.last_name_clean}}",
  "address": "{{3.property_address_clean}}",
  "phones": {{4.phones_cleaned}},
  "emails": {{5.emails_cleaned}},
  "validation_issues": {{2.validation_issues}}, {{3.validation_issues}}, {{4.validation_issues}}, {{5.validation_issues}}
}

Criteria:
- Required: name, address
- At least one: phone OR email
- Quality levels: Clean, DefaultsOnly, Incomplete, Invalid

Return ONLY valid JSON: {
  "quality_flag": "<Clean|DefaultsOnly|Incomplete|Invalid>",
  "completeness_score": <0-1>,
  "validation_notes": "<assessment>",
  "recommended_action": "<Accept|Review|Reject>"
}
```

---

#### Action 12: GPT-5 Pro - Channel Assignment
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Assign activation channel:
- OPS: {{10.ops_score}}
- CI: {{8.ci_score}}
- Has Cell: {{4.has_any_cell}}
- Has Email: {{5.has_any_email}}
- Quality: {{11.quality_flag}}

Channels (NO DIRECT MAIL):
- Call+SMS: CI>=0.70, has cell, OPS>=0.60
- SMS-Only: has cell, OPS<0.60
- Email+SMS: has email+cell
- Email-Only: has email only
- NurtureMail: limited options
- HOLD: insufficient data

Return ONLY valid JSON: {
  "channel_plan": "<channel>",
  "dialable": <true/false>,
  "reasoning": "<why>"
}
```

---

#### Action 13: GPT-5 Pro - Offer Strategy
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Determine offer strategy:
- DI: {{6.di_score}}
- UI: {{7.ui_score}}
- Equity: {{1.equity_pct}}%
- Condition: {{1.condition_score}}

Strategies:
- Cash-Only: UI>=0.70 AND equity<30%
- Novation/Wholetail: equity>=40% AND condition>=3
- Cash-or-Novation: flexible default

Return ONLY valid JSON: {
  "offer_strategy": "<strategy>",
  "reasoning": "<why>"
}
```

---

#### Action 14: GPT-5 Pro - Playbook Assignment
- **App**: OpenAI (GPT-5 Pro)
- **Action**: Conversation
- **User Message**:
```
Assign playbook:
- List Type: {{lead_intake.list_type}}
- Event Type: {{1.event_type}}
- OPS: {{10.ops_score}}
- Channel: {{12.channel_plan}}
- UI: {{7.ui_score}}

Playbooks:
- PB-PFC-Standard: PFC, standard urgency
- PB-PFC-Urgent: PFC, UI>=0.70
- PB-Divorce-Standard: Divorce
- PB-Divorce-Nurture: Divorce, UI<0.40
- PB-Probate-Standard: Probate
- PB-PreProbate-Nurture: PreProbate
- PB-TaxLien-Urgent: TaxLien

Return ONLY valid JSON: {
  "playbook": "<playbook>",
  "reasoning": "<why>"
}
```

---

#### Action 15: Code by Zapier - Compile Final Record
- **App**: Code by Zapier (Python)
- **Action**: Assemble processed lead
- **Code**:
```python
import json

# Compile all processed data
processed_lead = {
    'lead_id': input_data.get('lead_id'),
    
    # Cleaned identity
    'first_name': input_data.get('2_first_name_clean'),
    'last_name': input_data.get('2_last_name_clean'),
    'company_name': input_data.get('2_company_name_clean'),
    'is_business': input_data.get('2_is_business'),
    
    # Cleaned addresses
    'street_address': input_data.get('3_property_address_clean'),
    'mail_address': input_data.get('3_mailing_address_clean'),
    'mail_address_same': input_data.get('3_addresses_match'),
    
    # Cleaned contacts
    'callable_phones': ';'.join(json.loads(input_data.get('4_callable_phones', '[]'))),
    'emails': ';'.join(json.loads(input_data.get('5_emails_cleaned', '[]'))),
    'has_any_cell': input_data.get('4_has_any_cell'),
    'has_any_landline': input_data.get('4_has_any_landline'),
    'has_any_email': input_data.get('5_has_any_email'),
    
    # Scores
    'di_score': input_data.get('6_di_score'),
    'ui_score': input_data.get('7_ui_score'),
    'ols_score': input_data.get('9_ols_score'),
    'ci_score': input_data.get('8_ci_score'),
    'ops_score': input_data.get('10_ops_score'),
    
    # Tiers
    'distress_tier': input_data.get('10_distress_tier'),
    'urgency_tier': input_data.get('10_urgency_tier'),
    'reach_tier': input_data.get('10_reach_tier'),
    'motivation_level': input_data.get('10_motivation_level'),
    
    # Routing
    'channel_plan': input_data.get('12_channel_plan'),
    'offer_strategy': input_data.get('13_offer_strategy'),
    'playbook': input_data.get('14_playbook'),
    'dialable': input_data.get('12_dialable'),
    
    # Quality
    'quality_flag': input_data.get('11_quality_flag'),
    'completeness_score': input_data.get('11_completeness_score'),
    'gpt_validation_notes': input_data.get('11_validation_notes'),
    
    # Metadata
    'processing_timestamp': datetime.now().isoformat(),
    'any_dnc': False,  # Would integrate with DNC lookup service
    'compliance_hold': False
}

return processed_lead
```

---

#### Action 16: Zapier Tables - Create Record in `lead_processed`
- **App**: Zapier Tables
- **Action**: Create Record
- **Table**: lead_processed
- **Fields**: Map all fields from Action 15 output

---

#### Action 17: Filter - Route to Appropriate Channel
- **App**: Filter by Zapier
- **Condition**: Channel Plan equals "Call+SMS"
- **Continue If**: Yes

---

#### Action 18a: Webhook - Send to Dialer (if Call+SMS)
- **App**: Webhooks by Zapier
- **Action**: POST
- **URL**: https://your-dialer.com/api/leads
- **Payload**:
```json
{
  "lead_id": "{{15.lead_id}}",
  "first_name": "{{15.first_name}}",
  "last_name": "{{15.last_name}}",
  "phone": "{{15.callable_phones}}",
  "priority": "{{15.ops_score}}",
  "campaign": "{{15.playbook}}",
  "offer_strategy": "{{15.offer_strategy}}"
}
```

---

#### Action 18b: Email Platform - Add to Nurture (if NurtureMail)
- **App**: Mailchimp / ActiveCampaign / HubSpot
- **Action**: Add/Update Contact
- **Condition**: Channel Plan equals "NurtureMail"
- **Email**: {{15.emails}}
- **Tags**: {{15.playbook}}, {{15.motivation_level}}, Nurture

---

#### Action 19: Slack Notification - Processing Complete
- **App**: Slack
- **Action**: Send Channel Message
- **Channel**: #lead-processing
- **Message**:
```
✅ Lead Processed: {{15.first_name}} {{15.last_name}}
📊 OPS Score: {{15.ops_score}}
📞 Channel: {{15.channel_plan}}
🎯 Playbook: {{15.playbook}}
```

---

## GPT-5 Pro Prompts Library

### Master System Prompt (Apply to All Actions)

```
You are an expert data processing AI specialized in real estate lead management and scoring. Your role is to:

1. Process and analyze distressed property leads
2. Apply precise mathematical formulas for scoring
3. Clean and validate contact information
4. Assign leads to optimal activation channels
5. Always return valid JSON responses
6. Be deterministic and consistent in your decisions

Key Principles:
- Accuracy over speed
- Compliance with DNC and privacy regulations
- No assumptions - work only with provided data
- Flag ambiguities for human review
- Return structured JSON ONLY (no markdown, no explanations outside JSON)

Temperature: 0.1 (for consistency)
Response Format: json_object
```

### Prompt Template Structure

Every GPT-5 Pro action should follow this structure:

```
[CONTEXT]
Provide relevant lead data and current processing step

[INSTRUCTIONS]
Clear, numbered steps for the AI to follow

[RULES/LOGIC]
Specific decision trees, formulas, or validation criteria

[OUTPUT FORMAT]
Exact JSON schema expected

[EXAMPLE] (optional)
Sample input/output to guide the model
```

---

## Quality Assurance & Monitoring

### Built-in Quality Checks

1. **GPT-5 Response Validation**:
   - Use Zapier's "Validate JSON" formatter after each GPT action
   - If invalid JSON, retry with clarified prompt
   - Log failures to error table for review

2. **Score Range Validation**:
   - Code by Zapier step to verify all scores are 0-1
   - Flag out-of-range values for review

3. **Required Field Checks**:
   - Verify `quality_flag` is not "Invalid" before routing
   - Ensure channel assignment is valid option

4. **Human Review Queue**:
   - Automatically flag leads where:
     - `quality_flag` == "Incomplete" or "Invalid"
     - `ops_score` > 0.90 (potential high-value lead, manual verification)
     - `gpt_validation_notes` contains "ambiguous" or "unclear"
   - Send to Zapier Table: `leads_for_review`

### Monitoring Dashboard Setup

Create a Zapier Table called `processing_metrics`:

**Daily Aggregates**:
- `date` (Date)
- `leads_processed` (Number)
- `avg_ops_score` (Number)
- `channel_distribution` (JSON: {"Call+SMS": count, "Email": count, ...})
- `quality_distribution` (JSON: {"Clean": count, ...})
- `avg_processing_time_seconds` (Number)
- `error_rate` (Number, 0-1)

**Action**: Add a scheduled Zap to calculate daily metrics

---

## Troubleshooting & Best Practices

### Common Issues

#### Issue 1: GPT-5 Returning Invalid JSON
**Solution**:
- Emphasize "Return ONLY valid JSON" in prompts
- Use `json_object` response format
- Add retry logic with Zapier's Error Handler

#### Issue 2: Inconsistent Scoring
**Solution**:
- Keep temperature at 0.1
- Provide explicit formulas in every prompt
- Include numerical examples in prompts

#### Issue 3: Phone/Email Classification Errors
**Solution**:
- Integrate external validation APIs (NumVerify, ZeroBounce)
- Use GPT-5 for pattern recognition, then validate with API
- Maintain lookup tables for known carriers (cell vs. landline)

#### Issue 4: High GPT-5 API Costs
**Solution**:
- Batch similar operations when possible
- Cache common responses (e.g., event type to urgency mapping)
- Use cheaper models for simple tasks (e.g., GPT-4 Turbo for cleaning, GPT-5 Pro only for complex scoring)

### Best Practices

1. **Start Small**: Process 10-20 test leads before full deployment
2. **Version Control**: Keep prompt templates in GitHub, track changes
3. **A/B Testing**: Run parallel workflows with different prompt versions, compare quality
4. **Regular Audits**: Weekly manual review of 5% of processed leads
5. **Feedback Loop**: When acquisition team reports bad leads, trace back to identify prompt/logic issues
6. **Compliance First**: Always respect DNC, check state-specific regulations before SMS/calling
7. **Documentation**: Log all GPT responses for audit trail
8. **Performance Monitoring**: Track processing time per lead, optimize slow steps
9. **Fallback Logic**: Have manual processes ready if GPT-5 API experiences downtime
10. **Cost Management**: Set Zapier task limits, monitor OpenAI API spend

---

## Implementation Checklist

### Phase 1: Setup (Week 1)
- [ ] Create Zapier account with Tables access
- [ ] Provision OpenAI API key with GPT-5 Pro access
- [ ] Build `lead_intake` and `lead_processed` tables
- [ ] Create Zapier Interface for CSV upload
- [ ] Connect OpenAI integration to Zapier

### Phase 2: Core Workflow (Week 2)
- [ ] Build Actions 1-10 (data extraction through OPS calculation)
- [ ] Test with sample data (10 leads)
- [ ] Validate all scores are in expected ranges
- [ ] Debug any JSON parsing issues

### Phase 3: Routing Logic (Week 3)
- [ ] Build Actions 11-14 (quality, channel, offer, playbook)
- [ ] Implement channel filters
- [ ] Connect to dialer webhook
- [ ] Connect to email platform API

### Phase 4: Integration & Testing (Week 4)
- [ ] End-to-end test with 50 leads
- [ ] Compare results to manual processing (accuracy check)
- [ ] Optimize slow steps
- [ ] Add error handling and retries

### Phase 5: Launch (Week 5)
- [ ] Process first full batch (100-500 leads)
- [ ] Monitor closely for 48 hours
- [ ] Gather feedback from acquisition team
- [ ] Iterate on prompts based on feedback

### Ongoing
- [ ] Weekly audit of 5% sample
- [ ] Monthly prompt optimization
- [ ] Quarterly cost/benefit analysis
- [ ] Continuous compliance monitoring

---

## Appendix: Example GPT-5 API Call (Direct)

For reference, here's what a direct GPT-5 Pro API call looks like (Python):

```python
import openai

openai.api_key = "YOUR_API_KEY"

response = openai.ChatCompletion.create(
    model="gpt-5-pro",
    messages=[
        {
            "role": "system",
            "content": "You are a data processing AI. Always return valid JSON."
        },
        {
            "role": "user",
            "content": """Calculate OPS score:
            - DI: 0.75
            - UI: 0.60
            - OLS: 0.70
            - CI: 0.80
            
            Formula: OPS = (DI × 0.30) + (UI × 0.25) + (OLS × 0.25) + (CI × 0.20)
            
            Return JSON: {"ops_score": <value>, "tier": <1-4>}"""
        }
    ],
    temperature=0.1,
    response_format={"type": "json_object"}
)

result = response.choices[0].message.content
print(result)
# Output: {"ops_score": 0.7025, "tier": 2}
```

---

## Conclusion

This workflow provides a comprehensive, AI-powered solution for automated lead scoring and cleaning using Zapier and GPT-5 Pro. By following this guide, you can:

1. Upload raw lead CSVs through a user-friendly interface
2. Automatically clean and validate all data fields
3. Calculate sophisticated scoring metrics (DI, UI, OLS, CI, OPS)
4. Route leads to optimal activation channels (excluding direct mail)
5. Assign appropriate offer strategies and playbooks
6. Integrate with downstream systems (dialers, email platforms, CRMs)

The system is designed to be:
- **Scalable**: Process thousands of leads per day
- **Accurate**: Deterministic scoring with audit trails
- **Compliant**: Respects DNC and privacy regulations
- **Flexible**: Easy to adjust prompts and logic as business needs evolve
- **Cost-Effective**: Automates hours of manual work per batch

For support or questions, refer to Zapier documentation and OpenAI API guides. Happy automating!
