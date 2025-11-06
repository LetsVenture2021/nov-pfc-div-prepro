# Zapier AI Configuration Prompts

## How to Use This Document

This document contains ready-to-use prompts for Zapier AI to help you build the complete lead scoring and cleaning workflow. Copy and paste these prompts directly into Zapier AI's chat interface or workflow builder.

---

## Initial Setup Prompt

**Copy this prompt to Zapier AI to create the complete workflow:**

```
Create a Zapier workflow for automated lead scoring and cleaning with the following specifications:

WORKFLOW NAME: "Lead Processing Pipeline - GPT-5 Pro"

TRIGGER:
- New record added to Zapier Table called "lead_intake"

DATA STRUCTURE FOR lead_intake TABLE:
Create a table with these fields:
- lead_id (auto-generated unique ID)
- first_name (text)
- last_name (text)
- company_name (text, optional)
- street_address (text)
- city (text)
- state (text, 2 chars)
- zip (text)
- mail_street_address (text)
- mail_city (text)
- mail_state (text)
- mail_zip (text)
- phone_numbers (long text, semicolon-separated)
- email_addresses (long text, semicolon-separated)
- event_type (text: Auction, Divorce, Probate, TaxLien, PreProbate)
- event_date (date)
- data_source (text)
- imported_at (datetime, auto-set on creation)
- list_type (text: PFC, Divorce, Probate)
- estimated_equity_pct (number, 0-100)
- property_condition_score (number, 1-5)

WORKFLOW ACTIONS:
Build a series of actions that:
1. Extract and normalize the lead data
2. Use GPT-5 Pro to clean names, addresses, phones, and emails
3. Calculate scoring metrics: DI, UI, OLS, CI, OPS
4. Assign quality flags, channel plans, offer strategies, and playbooks
5. Store results in a "lead_processed" output table
6. Route leads to appropriate activation channels (NO DIRECT MAIL)

I will provide detailed prompts for each action in follow-up messages.
```

---

## Action-by-Action Configuration Prompts

### Action 1: Data Extraction

**Prompt to Zapier AI:**

```
Add a "Code by Zapier" action (Python) to extract and normalize lead data:

CODE:
import json
from datetime import datetime

lead = input_data
lead_id = lead.get('lead_id')

phones_raw = lead.get('phone_numbers', '')
phones_list = [p.strip() for p in phones_raw.split(';') if p.strip()]

emails_raw = lead.get('email_addresses', '')
emails_list = [e.strip().lower() for e in emails_raw.split(';') if e.strip()]

output = {
    'lead_id': lead_id,
    'first_name': lead.get('first_name', ''),
    'last_name': lead.get('last_name', ''),
    'company_name': lead.get('company_name', ''),
    'street_address': lead.get('street_address', ''),
    'city': lead.get('city', ''),
    'state': lead.get('state', ''),
    'zip': lead.get('zip', ''),
    'mail_street_address': lead.get('mail_street_address', ''),
    'mail_city': lead.get('mail_city', ''),
    'mail_state': lead.get('mail_state', ''),
    'mail_zip': lead.get('mail_zip', ''),
    'phones': phones_list,
    'emails': emails_list,
    'event_type': lead.get('event_type', ''),
    'event_date': lead.get('event_date', ''),
    'equity_pct': lead.get('estimated_equity_pct', 0),
    'condition_score': lead.get('property_condition_score', 3),
    'list_type': lead.get('list_type', ''),
    'current_date': datetime.now().strftime('%Y-%m-%d')
}

return output

LABEL this action: "Extract Lead Data"
```

---

### Action 2: Name Cleaning (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action with these settings:

MODEL: gpt-5-pro (or gpt-4-turbo if gpt-5-pro not available)
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a data cleaning specialist for real estate leads. Always return valid JSON. Be precise and consistent.

USER MESSAGE (use variables from previous step):
Clean and standardize this name:
First Name: "{first_name from step 1}"
Last Name: "{last_name from step 1}"
Company Name: "{company_name from step 1}"

Apply these rules:
1. Title case all names (proper capitalization)
2. Remove special characters except hyphens and apostrophes
3. Fix common OCR errors (e.g., "0" → "O" in names)
4. Detect and flag business entities vs. individuals
5. Standardize suffixes (Jr., Sr., II, III, IV)
6. Remove leading/trailing whitespace

Return ONLY valid JSON with this exact structure:
{
  "first_name_clean": "<cleaned first name>",
  "last_name_clean": "<cleaned last name>",
  "company_name_clean": "<cleaned company name>",
  "is_business": true or false,
  "validation_issues": ["list any issues found"]
}

LABEL this action: "Clean Names"
```

---

### Action 3: Address Validation (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are an address validation specialist. Always return valid JSON.

USER MESSAGE:
Validate and standardize these addresses:
Property Address: {street_address}, {city}, {state} {zip}
Mailing Address: {mail_street_address}, {mail_city}, {mail_state} {mail_zip}

Apply these rules:
1. Standardize street suffixes (St → Street, Ave → Avenue, Dr → Drive, Rd → Road, Blvd → Boulevard, Ln → Lane, Ct → Court)
2. Correct directionals (N, S, E, W, NE, NW, SE, SW)
3. Validate ZIP codes (5 or 9 digits, matches city/state patterns)
4. Flag PO Boxes in property address (invalid for properties)
5. Determine if mailing address matches property address
6. Validate state abbreviations (2-letter USPS codes)
7. Fix common misspellings in city names

Return ONLY valid JSON:
{
  "property_address_clean": "<full standardized property address>",
  "mailing_address_clean": "<full standardized mailing address>",
  "addresses_match": true or false,
  "property_has_pobox": true or false,
  "validation_issues": ["list any issues"],
  "confidence_score": <0.0 to 1.0>
}

LABEL this action: "Validate Addresses"
```

---

### Action 4: Phone Cleaning (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a phone number validation specialist. Always return valid JSON.

USER MESSAGE:
Process and classify these phone numbers:
Raw Input: {phones array from step 1}

Apply these rules:
1. Extract all valid 10-digit US phone numbers
2. Remove duplicates
3. Format consistently as: XXX-XXX-XXXX
4. Classify each as: Cell, Landline, VoIP, or Unknown
   - Cell phones typically have patterns with 5xx or 6xx in the exchange (middle 3 digits)
   - Landline typically starts with area codes + 2xx-4xx exchanges
   - If unsure, mark as Unknown
5. Flag obviously invalid numbers (all same digit, sequential, etc.)
6. Prioritize cell phones over landlines

Return ONLY valid JSON:
{
  "phones_cleaned": ["list of formatted valid phone numbers"],
  "phone_types": {"phone": "Cell|Landline|Unknown", ...},
  "has_any_cell": true or false,
  "has_any_landline": true or false,
  "callable_phones": ["list of all valid callable numbers"],
  "validation_issues": ["list any issues"]
}

LABEL this action: "Clean Phone Numbers"
```

---

### Action 5: Email Validation (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are an email validation specialist. Always return valid JSON.

USER MESSAGE:
Validate and clean these email addresses:
Raw Input: {emails array from step 1}

Apply these rules:
1. Extract valid email addresses matching standard format (user@domain.tld)
2. Remove exact duplicates
3. Lowercase all email addresses
4. Detect and flag disposable/temporary email providers (e.g., mailinator, tempmail, guerrillamail)
5. Check for common typos in popular domains:
   - gmial.com → gmail.com
   - yahooo.com → yahoo.com
   - hotmial.com → hotmail.com
   - outlok.com → outlook.com
6. Flag role-based emails (info@, admin@, sales@, support@) - less valuable for direct contact
7. Identify provider types (Gmail, Yahoo, Outlook, Corporate, ISP)

Return ONLY valid JSON:
{
  "emails_cleaned": ["list of valid, corrected email addresses"],
  "has_any_email": true or false,
  "disposable_emails": ["list of disposable email addresses found"],
  "role_based_emails": ["list of role-based addresses"],
  "validation_issues": ["list any issues"],
  "email_types": {"email": "Gmail|Yahoo|Outlook|Corporate|ISP", ...}
}

LABEL this action: "Validate Emails"
```

---

### Action 6: Calculate Distress Index (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a real estate lead scoring specialist. Use exact mathematical formulas. Always return valid JSON.

USER MESSAGE:
Calculate the Distress Index (DI) for this lead:
- Estimated Equity Percentage: {equity_pct}%
- Event Type: {event_type}
- Property Condition Score: {condition_score} (on 1-5 scale where 1=Poor, 5=Excellent)

Apply this exact formula:

Step 1: Normalize equity
EstEquityPct_Norm = MIN(EstimatedEquity% / 50, 1.0)
This means: divide the equity percentage by 50, but cap the result at 1.0
Examples: 25% equity → 0.5, 50% equity → 1.0, 75% equity → 1.0

Step 2: Assign Event Urgency Score based on event type:
- Auction or Foreclosure: 1.0 (highest distress)
- TaxLien or Tax Lien: 0.85
- Divorce: 0.60
- Probate: 0.50
- PreProbate or Pre-Probate: 0.30
- Unknown or Other: 0.40

Step 3: Calculate Condition Penalty
ConditionPenalty = (5 - ConditionScore) / 5
Examples: Condition 1 (poor) → penalty 0.8, Condition 5 (excellent) → penalty 0.0

Step 4: Calculate DI
DI = (EstEquityPct_Norm × 0.35) + (EventUrgency × 0.40) + (ConditionPenalty × 0.25)

Return ONLY valid JSON:
{
  "di_score": <calculated value, rounded to 4 decimal places>,
  "equity_normalized": <calculated value>,
  "event_urgency": <assigned value>,
  "condition_penalty": <calculated value>,
  "reasoning": "<brief 1-sentence explanation of the score>"
}

LABEL this action: "Calculate DI Score"
```

---

### Action 7: Calculate Urgency Index (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a real estate lead scoring specialist. Use exact date calculations. Always return valid JSON.

USER MESSAGE:
Calculate the Urgency Index (UI) for this lead:
- Event Type: {event_type}
- Event Date: {event_date}
- Today's Date: {current_date}

Apply this exact logic:

Step 1: Calculate DaysUntilEvent
If event_date is provided: DaysUntilEvent = event_date - today
If event_date is empty or "Unknown": DaysUntilEvent = 999 (placeholder for no date)

Step 2: Assign TimeDecay Score based on DaysUntilEvent:
- 0 to 30 days: 1.0 (maximum urgency)
- 31 to 60 days: 0.70
- 61 to 90 days: 0.45
- 91 to 180 days: 0.25
- 181+ days: 0.10
- No date/Unknown (999): 0.05

Step 3: Assign EventWeight based on event type:
- Auction or Foreclosure: 1.0 (date is critical)
- TaxLien or Tax Lien: 0.85
- Divorce: 0.70 (dates less rigid)
- Probate: 0.60
- PreProbate or Pre-Probate: 0.40 (very flexible timeline)
- Unknown or Other: 0.50

Step 4: Calculate UI
UI = TimeDecay × EventWeight

Return ONLY valid JSON:
{
  "ui_score": <calculated value, rounded to 4 decimal places>,
  "days_until_event": <calculated days or "unknown">,
  "time_decay": <assigned value>,
  "event_weight": <assigned value>,
  "reasoning": "<brief 1-sentence explanation>"
}

LABEL this action: "Calculate UI Score"
```

---

### Action 8: Calculate Contact-ability Index (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a lead contact-ability specialist. Use exact formulas. Always return valid JSON.

USER MESSAGE:
Calculate the Contact-ability Index (CI) for this lead:
- Has Any Cell Phone: {has_any_cell}
- Has Any Landline: {has_any_landline}
- Has Any Email: {has_any_email}
- Mailing Address Matches Property: {addresses_match}

Apply this exact formula:

Step 1: Calculate PhoneScore
- If has_any_cell is true: PhoneScore = 1.0
- Else if has_any_landline is true: PhoneScore = 0.60
- Else (no phone): PhoneScore = 0.0

Step 2: Calculate EmailScore
- If has_any_email is true: EmailScore = 1.0
- Else (no email): EmailScore = 0.0

Step 3: Calculate AddressScore
- If addresses_match is true (mailing = property): AddressScore = 1.0
- Else if mailing address exists but different: AddressScore = 0.70
- Else (no valid mailing address): AddressScore = 0.0

Step 4: Calculate CI
CI = (PhoneScore × 0.40) + (EmailScore × 0.25) + (AddressScore × 0.20) + 0.15
The 0.15 is a base score for having basic information

Note: If lead is on Do Not Call (DNC) list, final CI should be multiplied by 0.0, but we handle that later.

Return ONLY valid JSON:
{
  "ci_score": <calculated value, rounded to 4 decimal places>,
  "phone_score": <calculated value>,
  "email_score": <calculated value>,
  "address_score": <calculated value>,
  "reasoning": "<brief 1-sentence explanation>"
}

LABEL this action: "Calculate CI Score"
```

---

### Action 9: Calculate Offer-Likelihood Score (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a real estate offer prediction specialist. Use exact formulas. Always return valid JSON.

USER MESSAGE:
Calculate the Offer-Likelihood Score (OLS) for this lead:
- Distress Index (DI): {di_score from step 6}
- Urgency Index (UI): {ui_score from step 7}
- Estimated Equity Percentage: {equity_pct}
- Property Condition Score: {condition_score}

Apply this exact logic:

Step 1: Determine MarketFactor
Based on property characteristics:
- If equity >= 40% AND condition >= 4: MarketFactor = 0.70 (strong property, many options)
- Else if equity >= 30% AND condition >= 3: MarketFactor = 0.60 (decent property)
- Else if equity >= 20% AND condition <= 2: MarketFactor = 0.80 (poor condition, high distress)
- Else if equity < 20%: MarketFactor = 0.90 (underwater or low equity, very motivated)
- Default: MarketFactor = 0.60

Step 2: Calculate OLS
OLS = (DI × 0.50) + (UI × 0.30) + (MarketFactor × 0.20)

Return ONLY valid JSON:
{
  "ols_score": <calculated value, rounded to 4 decimal places>,
  "market_factor": <assigned value>,
  "reasoning": "<brief 1-sentence explanation>"
}

LABEL this action: "Calculate OLS Score"
```

---

### Action 10: Calculate Overall Profitability Score (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a real estate lead scoring specialist. Calculate final scores and assign tiers. Always return valid JSON.

USER MESSAGE:
Calculate the Overall Profitability Score (OPS) and assign tiers:
- Distress Index (DI): {di_score from step 6}
- Urgency Index (UI): {ui_score from step 7}
- Offer-Likelihood Score (OLS): {ols_score from step 9}
- Contact-ability Index (CI): {ci_score from step 8}

Apply this exact formula:

Step 1: Calculate OPS
OPS = (DI × 0.30) + (UI × 0.25) + (OLS × 0.25) + (CI × 0.20)

Step 2: Assign overall tier based on OPS:
- 0.80 to 1.00: Tier 1
- 0.60 to 0.79: Tier 2
- 0.40 to 0.59: Tier 3
- 0.00 to 0.39: Tier 4

Step 3: Assign component tiers (H=High, M=Medium, L=Low):
Distress Tier:
- DI >= 0.70: H (High Distress)
- DI >= 0.40: M (Medium Distress)
- DI < 0.40: L (Low Distress)

Urgency Tier:
- UI >= 0.70: H (High Urgency)
- UI >= 0.40: M (Medium Urgency)
- UI < 0.40: L (Low Urgency)

Reach Tier (based on CI):
- CI >= 0.70: H (High Reachability)
- CI >= 0.40: M (Medium Reachability)
- CI < 0.40: L (Low Reachability)

Step 4: Assign Motivation Level:
- If Distress=H AND Urgency=H: "High"
- Else if Distress>=M OR Urgency>=M: "Medium"
- Else: "Low"

Return ONLY valid JSON:
{
  "ops_score": <calculated value, rounded to 4 decimal places>,
  "tier": <1, 2, 3, or 4>,
  "tier_label": "<Hot|Warm|Nurture|Low>",
  "distress_tier": "<H|M|L>",
  "urgency_tier": "<H|M|L>",
  "reach_tier": "<H|M|L>",
  "motivation_level": "<High|Medium|Low>",
  "reasoning": "<brief comprehensive explanation of the overall score>"
}

LABEL this action: "Calculate OPS Score and Tiers"
```

---

### Action 11: Data Quality Assessment (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a data quality specialist. Assess lead completeness and usability. Always return valid JSON.

USER MESSAGE:
Assess the data quality and completeness for this lead:

Lead Identity:
- Name: {first_name_clean} {last_name_clean}
- Company: {company_name_clean}
- Is Business: {is_business}

Addresses:
- Property Address: {property_address_clean}
- Mailing Address: {mailing_address_clean}
- Confidence: {confidence_score from step 3}

Contact Methods:
- Phones: {phones_cleaned from step 4}
- Emails: {emails_cleaned from step 5}
- Has Cell: {has_any_cell}
- Has Email: {has_any_email}

Validation Issues Collected:
- Name issues: {validation_issues from step 2}
- Address issues: {validation_issues from step 3}
- Phone issues: {validation_issues from step 4}
- Email issues: {validation_issues from step 5}

Apply these quality criteria:

1. Required fields present:
   - Must have: First Name, Last Name, Property Address (city, state, zip)
   - Preferred: At least one contact method (phone OR email)

2. Data completeness scoring:
   - 1.0: All required fields + multiple contact methods + no issues
   - 0.8: All required fields + one contact method + minor issues
   - 0.6: Required fields present + limited contact + some issues
   - 0.4: Missing some required fields OR major validation issues
   - 0.2: Multiple missing fields, serious issues

3. Quality flag assignment:
   - "Clean": Completeness >= 0.8, no serious validation issues
   - "DefaultsOnly": Completeness >= 0.6, minor issues acceptable
   - "Incomplete": Completeness >= 0.4, usable but limited
   - "Invalid": Completeness < 0.4 OR serious validation failures

4. Recommended action:
   - "Accept": Quality flag is Clean or DefaultsOnly
   - "Review": Quality flag is Incomplete, manual review needed
   - "Reject": Quality flag is Invalid, do not process

Return ONLY valid JSON:
{
  "quality_flag": "<Clean|DefaultsOnly|Incomplete|Invalid>",
  "completeness_score": <0.0 to 1.0, rounded to 2 decimal places>,
  "missing_fields": ["list of critical missing fields"],
  "validation_notes": "<detailed 2-3 sentence assessment>",
  "recommended_action": "<Accept|Review|Reject>"
}

LABEL this action: "Assess Data Quality"
```

---

### Action 12: Channel Assignment (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a marketing channel specialist. Assign leads to optimal activation channels. NO DIRECT MAIL options. Always return valid JSON.

USER MESSAGE:
Determine the optimal activation channel for this lead:
- Overall Profitability Score (OPS): {ops_score from step 10}
- Contact-ability Index (CI): {ci_score from step 8}
- Has Any Cell Phone: {has_any_cell from step 4}
- Has Any Landline: {has_any_landline from step 4}
- Has Any Email: {has_any_email from step 5}
- Quality Flag: {quality_flag from step 11}
- Recommended Action: {recommended_action from step 11}

Available activation channels (NO DIRECT MAIL):
1. Call+SMS: Outbound calling + text messaging (high-touch)
2. SMS-Only: Text messaging campaigns only (medium-touch)
3. Email+SMS: Email campaigns supplemented with texts
4. Email-Only: Email campaigns only
5. NurtureMail: Long-term email nurture sequences (low-touch)
6. HOLD: Insufficient data, manual review required
7. REJECT: Do not process

Apply this decision tree logic:

IF recommended_action == "Reject" OR quality_flag == "Invalid":
  → Channel = "REJECT", Dialable = false

ELIF quality_flag == "Incomplete":
  → Channel = "HOLD", Dialable = false

ELIF CI >= 0.70 AND has_any_cell == true:
  IF OPS >= 0.60:
    → Channel = "Call+SMS", Dialable = true (high-value, highly reachable)
  ELSE:
    → Channel = "SMS-Only", Dialable = false (reachable but lower value)

ELIF CI >= 0.50 AND has_any_email == true:
  IF has_any_cell == true AND OPS >= 0.60:
    → Channel = "Email+SMS", Dialable = false
  ELSE:
    → Channel = "Email-Only", Dialable = false

ELIF has_any_email == true:
  → Channel = "NurtureMail", Dialable = false (limited contact options)

ELSE:
  → Channel = "HOLD", Dialable = false (no valid contact methods)

Return ONLY valid JSON:
{
  "channel_plan": "<selected channel from list above>",
  "dialable": true or false,
  "reasoning": "<brief 1-2 sentence explanation of why this channel was chosen>",
  "alternate_channels": ["<list 1-2 backup channels if primary fails>"],
  "compliance_notes": "<any compliance considerations, or 'None'>"
}

LABEL this action: "Assign Activation Channel"
```

---

### Action 13: Offer Strategy Assignment (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a real estate offer strategy specialist. Determine the best acquisition approach. Always return valid JSON.

USER MESSAGE:
Determine the optimal offer strategy for this lead:
- Distress Index (DI): {di_score from step 6}
- Urgency Index (UI): {ui_score from step 7}
- Estimated Equity Percentage: {equity_pct}
- Property Condition Score: {condition_score}
- Event Type: {event_type}
- Overall Profitability Score (OPS): {ops_score from step 10}

Available offer strategies:
1. Cash-Only: Quick cash offers for urgent, distressed situations
2. Novation/Wholetail: Creative financing, seller-stays options, subject-to deals
3. Cash-or-Novation: Flexible approach, present both options

Apply this decision logic:

IF UI >= 0.70 AND equity_pct < 30%:
  → Strategy = "Cash-Only"
  Reasoning: High urgency + low equity = need quick cash exit

ELIF equity_pct >= 40% AND condition_score >= 3:
  → Strategy = "Novation/Wholetail"
  Reasoning: Good property with equity allows creative financing

ELIF DI >= 0.70 AND UI >= 0.60:
  → Strategy = "Cash-Only"
  Reasoning: High distress + high urgency = immediate cash needed

ELIF condition_score <= 2:
  → Strategy = "Cash-Only"
  Reasoning: Poor condition properties best suited for cash offers

ELSE:
  → Strategy = "Cash-or-Novation"
  Reasoning: Flexible approach allows tailoring to seller needs

Return ONLY valid JSON:
{
  "offer_strategy": "<Cash-Only|Novation/Wholetail|Cash-or-Novation>",
  "reasoning": "<brief 1-2 sentence explanation>",
  "talking_points": [
    "<key point 1 for acquisition team>",
    "<key point 2 for acquisition team>",
    "<key point 3 for acquisition team>"
  ],
  "estimated_close_probability": "<High|Medium|Low>"
}

LABEL this action: "Assign Offer Strategy"
```

---

### Action 14: Playbook Assignment (GPT-5 Pro)

**Prompt to Zapier AI:**

```
Add an "OpenAI (ChatGPT)" action:

MODEL: gpt-5-pro
TEMPERATURE: 0.1
RESPONSE FORMAT: json_object

SYSTEM MESSAGE:
You are a workflow playbook specialist. Assign the appropriate engagement playbook. Always return valid JSON.

USER MESSAGE:
Assign the appropriate engagement playbook for this lead:
- List Type: {list_type}
- Event Type: {event_type}
- Overall Profitability Score (OPS): {ops_score from step 10}
- Urgency Index (UI): {ui_score from step 7}
- Channel Plan: {channel_plan from step 12}
- Offer Strategy: {offer_strategy from step 13}
- Motivation Level: {motivation_level from step 10}

Available playbooks:
1. PB-PFC-Standard: Standard pre-foreclosure workflow (moderate urgency)
2. PB-PFC-Urgent: High-urgency pre-foreclosure (auction within 30 days)
3. PB-Divorce-Standard: Divorce-motivated sellers (standard timeline)
4. PB-Divorce-Nurture: Lower-urgency divorce cases (long-term cultivation)
5. PB-Probate-Standard: Active probate cases (court-driven timeline)
6. PB-PreProbate-Nurture: Pre-probate leads (early stage, long-term)
7. PB-TaxLien-Urgent: Tax lien with near-term sale date
8. PB-Generic-Standard: Other distressed situations

Apply this decision logic:

Step 1: Match primary category based on List Type and Event Type:
- If list_type or event_type contains "PFC" or "Foreclosure" or "Auction": Category = PFC
- If contains "Divorce": Category = Divorce
- If contains "Probate": Category = Probate
- If contains "PreProbate" or "Pre-Probate": Category = PreProbate
- If contains "TaxLien" or "Tax Lien": Category = TaxLien
- Else: Category = Generic

Step 2: Determine urgency variant:
- If UI >= 0.70 OR motivation_level == "High": Variant = Urgent or Standard
- If UI < 0.40 OR motivation_level == "Low": Variant = Nurture
- Else: Variant = Standard

Step 3: Combine to select playbook:
- PFC + Urgent/High: PB-PFC-Urgent
- PFC + Standard: PB-PFC-Standard
- Divorce + Nurture: PB-Divorce-Nurture
- Divorce + other: PB-Divorce-Standard
- Probate: PB-Probate-Standard
- PreProbate: PB-PreProbate-Nurture
- TaxLien: PB-TaxLien-Urgent
- Generic: PB-Generic-Standard

Return ONLY valid JSON:
{
  "playbook": "<selected playbook code>",
  "reasoning": "<brief 1-2 sentence explanation>",
  "key_actions": [
    "<main action 1 in this playbook>",
    "<main action 2>",
    "<main action 3>"
  ],
  "timeline": "<expected duration: e.g., '2-4 weeks' or '3-6 months'>",
  "success_indicators": ["<metric 1>", "<metric 2>"]
}

LABEL this action: "Assign Playbook"
```

---

### Action 15: Compile Final Record (Code)

**Prompt to Zapier AI:**

```
Add a "Code by Zapier" action (Python) to compile all processed data:

CODE:
import json
from datetime import datetime

# Compile all processed data into final record
processed_lead = {
    'lead_id': input_data.get('lead_id'),
    
    # Original data
    'data_source': input_data.get('data_source'),
    'list_type': input_data.get('list_type'),
    'imported_at': input_data.get('imported_at'),
    
    # Cleaned identity
    'first_name': input_data.get('first_name_clean'),
    'last_name': input_data.get('last_name_clean'),
    'company_name': input_data.get('company_name_clean'),
    'is_business': input_data.get('is_business', False),
    
    # Cleaned addresses
    'street_address': input_data.get('property_address_clean'),
    'city': input_data.get('city'),
    'state': input_data.get('state'),
    'zip': input_data.get('zip'),
    'mail_street_address': input_data.get('mailing_address_clean'),
    'mail_address_same': input_data.get('addresses_match', False),
    
    # Cleaned contacts
    'callable_phones': ';'.join(input_data.get('callable_phones', [])),
    'emails': ';'.join(input_data.get('emails_cleaned', [])),
    'has_any_cell': input_data.get('has_any_cell', False),
    'has_any_landline': input_data.get('has_any_landline', False),
    'has_any_email': input_data.get('has_any_email', False),
    
    # Event data
    'event_type': input_data.get('event_type'),
    'event_date': input_data.get('event_date'),
    'days_until_event': input_data.get('days_until_event'),
    'estimated_equity_pct': input_data.get('equity_pct'),
    'property_condition_score': input_data.get('condition_score'),
    
    # Scores
    'di_score': float(input_data.get('di_score', 0)),
    'ui_score': float(input_data.get('ui_score', 0)),
    'ols_score': float(input_data.get('ols_score', 0)),
    'ci_score': float(input_data.get('ci_score', 0)),
    'ops_score': float(input_data.get('ops_score', 0)),
    'rank_ops': 0,  # Will be calculated in batch ranking process
    
    # Tiers
    'distress_tier': input_data.get('distress_tier'),
    'urgency_tier': input_data.get('urgency_tier'),
    'reach_tier': input_data.get('reach_tier'),
    'motivation_level': input_data.get('motivation_level'),
    
    # Routing
    'channel_plan': input_data.get('channel_plan'),
    'offer_strategy': input_data.get('offer_strategy'),
    'playbook': input_data.get('playbook'),
    'dialable': input_data.get('dialable', False),
    
    # Quality
    'quality_flag': input_data.get('quality_flag'),
    'completeness_score': float(input_data.get('completeness_score', 0)),
    'gpt_validation_notes': input_data.get('validation_notes', ''),
    
    # Metadata
    'processing_timestamp': datetime.now().isoformat(),
    'any_dnc': False,  # Would integrate with real DNC lookup service
    'compliance_hold': False
}

return processed_lead

LABEL this action: "Compile Final Record"
```

---

### Action 16: Store in Output Table

**Prompt to Zapier AI:**

```
Add a "Zapier Tables" action to store the processed lead:

ACTION: Create Record
TABLE: lead_processed (create this table if it doesn't exist)

TABLE STRUCTURE for lead_processed:
Copy all fields from lead_intake table, plus add these additional fields:
- ops_score (number)
- rank_ops (number)
- di_score (number)
- ui_score (number)
- ols_score (number)
- ci_score (number)
- distress_tier (text: H/M/L)
- urgency_tier (text: H/M/L)
- reach_tier (text: H/M/L)
- motivation_level (text: High/Medium/Low)
- channel_plan (text)
- offer_strategy (text)
- playbook (text)
- callable_phones (long text)
- has_any_cell (boolean)
- has_any_landline (boolean)
- has_any_email (boolean)
- any_dnc (boolean)
- compliance_hold (boolean)
- dialable (boolean)
- quality_flag (text)
- completeness_score (number)
- gpt_validation_notes (long text)
- processing_timestamp (datetime)
- is_business (boolean)

FIELD MAPPINGS:
Map all fields from the previous step (Action 15) output to the corresponding table fields.

LABEL this action: "Store Processed Lead"
```

---

### Action 17: Route to Activation Channels

**Prompt to Zapier AI:**

```
Add a "Paths by Zapier" action to route leads to different activation channels:

CREATE PATHS BASED ON: {channel_plan from step 16}

PATH A: Call+SMS Channel
- Rule: channel_plan equals "Call+SMS"
- Action: Webhook POST to dialer system
  URL: [Your dialer webhook URL]
  Method: POST
  Data:
  {
    "lead_id": "{lead_id}",
    "first_name": "{first_name}",
    "last_name": "{last_name}",
    "phone": "{callable_phones}",
    "priority_score": "{ops_score}",
    "campaign": "{playbook}",
    "offer_type": "{offer_strategy}",
    "notes": "{gpt_validation_notes}"
  }

PATH B: SMS-Only Channel
- Rule: channel_plan equals "SMS-Only"
- Action: Add to SMS platform (e.g., Twilio, EZ Texting)
  
PATH C: Email Channels (Email+SMS, Email-Only, NurtureMail)
- Rule: channel_plan contains "Email" OR equals "NurtureMail"
- Action: Add to email marketing platform (Mailchimp, ActiveCampaign, etc.)
  Email: {emails}
  Tags: {playbook}, {motivation_level}, {list_type}
  Custom Fields:
    - ops_score: {ops_score}
    - offer_strategy: {offer_strategy}

PATH D: Hold for Review
- Rule: channel_plan equals "HOLD"
- Action: Create record in "leads_for_review" table
  Flag for manual review

PATH E: Reject
- Rule: channel_plan equals "REJECT"
- Action: No routing, log to rejected leads table

LABEL this action: "Route to Activation Channels"
```

---

### Action 18: Send Notification

**Prompt to Zapier AI:**

```
Add a "Slack" (or email) action to send processing notification:

APP: Slack
ACTION: Send Channel Message
CHANNEL: #lead-processing (or your preferred channel)

MESSAGE:
✅ *Lead Processed Successfully*

*Lead Info:*
• Name: {first_name} {last_name}
• Lead ID: {lead_id}
• List Type: {list_type}
• Event: {event_type}

*Scores:*
• OPS: {ops_score} (Tier {tier} - {tier_label})
• Distress: {distress_tier} | Urgency: {urgency_tier} | Reach: {reach_tier}
• Motivation: {motivation_level}

*Routing:*
• Channel: {channel_plan}
• Dialable: {dialable}
• Offer Strategy: {offer_strategy}
• Playbook: {playbook}

*Quality:*
• Quality Flag: {quality_flag}
• Completeness: {completeness_score}

---
Processed at: {processing_timestamp}

LABEL this action: "Send Success Notification"
```

---

## Complete Setup Prompt (Single Paste)

**For those who want to paste one comprehensive prompt:**

```
Build a complete Zapier workflow called "Lead Processing Pipeline - GPT-5 Pro" with these specifications:

TABLES NEEDED:
1. lead_intake (input table with fields: lead_id, first_name, last_name, company_name, street_address, city, state, zip, mail_street_address, mail_city, mail_state, mail_zip, phone_numbers, email_addresses, event_type, event_date, data_source, imported_at, list_type, estimated_equity_pct, property_condition_score)

2. lead_processed (output table with all fields from lead_intake PLUS: ops_score, rank_ops, di_score, ui_score, ols_score, ci_score, distress_tier, urgency_tier, reach_tier, motivation_level, channel_plan, offer_strategy, playbook, callable_phones, has_any_cell, has_any_landline, has_any_email, any_dnc, compliance_hold, dialable, quality_flag, completeness_score, gpt_validation_notes, processing_timestamp, is_business)

WORKFLOW STRUCTURE:
Trigger: New record in lead_intake table

Action Flow:
1. Extract and normalize lead data (Python code)
2. Clean names with GPT-5 Pro
3. Validate addresses with GPT-5 Pro
4. Clean phone numbers with GPT-5 Pro
5. Validate emails with GPT-5 Pro
6. Calculate Distress Index (DI) with GPT-5 Pro
7. Calculate Urgency Index (UI) with GPT-5 Pro
8. Calculate Contact-ability Index (CI) with GPT-5 Pro
9. Calculate Offer-Likelihood Score (OLS) with GPT-5 Pro
10. Calculate Overall Profitability Score (OPS) with GPT-5 Pro
11. Assess data quality with GPT-5 Pro
12. Assign activation channel with GPT-5 Pro (NO DIRECT MAIL)
13. Assign offer strategy with GPT-5 Pro
14. Assign playbook with GPT-5 Pro
15. Compile final record (Python code)
16. Store in lead_processed table
17. Route to activation channels based on channel_plan (Call+SMS, SMS-Only, Email, NurtureMail, HOLD, REJECT)
18. Send Slack notification

GPT-5 PRO SETTINGS FOR ALL AI ACTIONS:
- Model: gpt-5-pro (or gpt-4-turbo)
- Temperature: 0.1
- Response Format: json_object
- System Message: "You are a data processing specialist. Always return valid JSON. Be precise and consistent."

Build this workflow with all actions properly connected and configured.
```

---

## Testing and Validation Prompts

### Test Data Upload Prompt

```
I need to test this workflow. Create a test record in the lead_intake table with these values:

first_name: John
last_name: Smith
street_address: 123 Main Street
city: Houston
state: TX
zip: 77001
mail_street_address: 123 Main Street
mail_city: Houston
mail_state: TX
mail_zip: 77001
phone_numbers: 832-555-1234;281-555-5678
email_addresses: john.smith@gmail.com
event_type: Auction
event_date: [30 days from today]
list_type: PFC
estimated_equity_pct: 35
property_condition_score: 3
data_source: test_batch_001

Then trigger the workflow and show me the results.
```

---

## Modification Prompts

### Add DNC Integration

```
Add a new action after "Clean Phone Numbers" (step 4) to check phones against Do Not Call registry:

Use a webhook to check DNC status:
URL: https://your-dnc-service.com/api/check
Method: POST
Body: {"phones": {phones_cleaned from step 4}}

Then update the CI calculation (step 8) to multiply CI by 0.0 if any_dnc is true.

Update the Channel Assignment (step 12) to automatically route to "NurtureMail" if any_dnc is true.
```

### Add Email Verification API

```
Add a new action after "Validate Emails" (step 5) to verify emails using ZeroBounce or similar:

Use a webhook:
URL: https://api.zerobounce.net/v2/validate
Method: GET
Parameters: api_key=[YOUR_API_KEY], email={each email from step 5}

Update the email validation results with deliverability scores.
Filter out invalid/bounce emails before storing in lead_processed table.
```

---

This completes the Zapier AI prompts document. Use these prompts to configure your workflow step-by-step or paste the complete setup prompt for faster deployment.
