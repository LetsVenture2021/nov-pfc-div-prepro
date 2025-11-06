# Zapier Workflow Visual Diagrams

## Complete Workflow Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         LEAD PROCESSING PIPELINE                         │
│                      Powered by Zapier + GPT-5 Pro                       │
└─────────────────────────────────────────────────────────────────────────┘

    ┌──────────────────┐
    │   CSV Upload     │  ← User uploads lead file via Zapier Interface
    │   (Interface)    │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │  Zapier Table:   │  ← Raw leads stored
    │   lead_intake    │
    └────────┬─────────┘
             │
             ▼
    ╔════════════════════╗
    ║  PROCESSING FLOW   ║
    ╚════════════════════╝

┌──────────────────────────────────────────────────────────────────────────┐
│ PHASE 1: DATA EXTRACTION & NORMALIZATION                                 │
└──────────────────────────────────────────────────────────────────────────┘

    [Action 1: Extract Data]
    • Parse phone numbers (split by ;)
    • Parse email addresses (split by ;)
    • Normalize field names
    • Calculate current date
             │
             ▼

┌──────────────────────────────────────────────────────────────────────────┐
│ PHASE 2: DATA CLEANING (GPT-5 Pro)                                       │
└──────────────────────────────────────────────────────────────────────────┘

    [Action 2: Clean Names] ──────────────┐
    GPT-5 Pro analyzes:                   │
    • Fix capitalization                  │
    • Remove special chars                │
    • Detect business vs. individual      │
    • Standardize suffixes                │
             │                            │
             ▼                            │
    [Action 3: Validate Addresses] ──────┤
    GPT-5 Pro analyzes:                   │
    • Standardize street suffixes         │  All cleaning
    • Validate ZIP codes                  │  happens in
    • Flag PO Boxes                       │  parallel
    • Check address matching              │  with GPT-5
             │                            │  Pro
             ▼                            │
    [Action 4: Clean Phones] ────────────┤
    GPT-5 Pro analyzes:                   │
    • Extract 10-digit numbers            │
    • Classify Cell vs. Landline          │
    • Remove duplicates                   │
    • Format consistently                 │
             │                            │
             ▼                            │
    [Action 5: Validate Emails] ─────────┘
    GPT-5 Pro analyzes:
    • Check valid format
    • Fix common typos
    • Flag disposable emails
    • Detect role-based addresses
             │
             ▼

┌──────────────────────────────────────────────────────────────────────────┐
│ PHASE 3: SCORING CALCULATIONS (GPT-5 Pro)                                │
└──────────────────────────────────────────────────────────────────────────┘

    [Action 6: Calculate DI] ─────────────┐
    Distress Index (0-1)                  │
    Formula: (Equity×0.35) +              │
             (EventUrg×0.40) +            │
             (CondPen×0.25)               │
             │                            │
             ▼                            │
    [Action 7: Calculate UI] ─────────────┤
    Urgency Index (0-1)                   │
    Formula: TimeDecay × EventWeight      │  All scores
             │                            │  calculated
             ▼                            │  by GPT-5
    [Action 8: Calculate CI] ─────────────┤  Pro using
    Contact-ability Index (0-1)           │  exact
    Formula: (Phone×0.40) +               │  formulas
             (Email×0.25) +               │
             (Addr×0.20) +                │
             Base 0.15                    │
             │                            │
             ▼                            │
    [Action 9: Calculate OLS] ────────────┤
    Offer-Likelihood Score (0-1)          │
    Formula: (DI×0.50) +                  │
             (UI×0.30) +                  │
             (MktFactor×0.20)             │
             │                            │
             ▼                            │
    [Action 10: Calculate OPS] ───────────┘
    Overall Profitability (0-1)
    Formula: (DI×0.30) +
             (UI×0.25) +
             (OLS×0.25) +
             (CI×0.20)
    
    Also assigns:
    • Tier (1-4)
    • Distress Tier (H/M/L)
    • Urgency Tier (H/M/L)
    • Reach Tier (H/M/L)
    • Motivation Level
             │
             ▼

┌──────────────────────────────────────────────────────────────────────────┐
│ PHASE 4: QUALITY & ROUTING (GPT-5 Pro)                                   │
└──────────────────────────────────────────────────────────────────────────┘

    [Action 11: Quality Assessment] ──────┐
    GPT-5 Pro determines:                 │
    • Quality Flag                        │
      (Clean/DefaultsOnly/                │
       Incomplete/Invalid)                │
    • Completeness Score (0-1)            │
    • Recommended Action                  │
             │                            │
             ▼                            │
    [Action 12: Channel Assignment] ──────┤
    GPT-5 Pro assigns:                    │  Routing
    • Call+SMS                            │  logic
    • SMS-Only                            │  applied
    • Email+SMS                           │  by GPT-5
    • Email-Only                          │  Pro
    • NurtureMail                         │
    • HOLD/REJECT                         │
             │                            │
             ▼                            │
    [Action 13: Offer Strategy] ──────────┤
    GPT-5 Pro assigns:                    │
    • Cash-Only                           │
    • Novation/Wholetail                  │
    • Cash-or-Novation                    │
             │                            │
             ▼                            │
    [Action 14: Playbook Assignment] ─────┘
    GPT-5 Pro assigns:
    • PB-PFC-Standard
    • PB-PFC-Urgent
    • PB-Divorce-Standard
    • PB-Divorce-Nurture
    • PB-Probate-Standard
    • PB-PreProbate-Nurture
    • PB-TaxLien-Urgent
             │
             ▼

┌──────────────────────────────────────────────────────────────────────────┐
│ PHASE 5: STORAGE & DISTRIBUTION                                          │
└──────────────────────────────────────────────────────────────────────────┘

    [Action 15: Compile Record]
    Python code assembles all
    processed data into final
    lead object
             │
             ▼
    ┌──────────────────┐
    │  Zapier Table:   │  ← Processed leads stored
    │ lead_processed   │
    └────────┬─────────┘
             │
             ▼
    [Action 17: Route by Channel]
             │
        ┌────┴────┬────────┬──────────┬──────────┐
        │         │        │          │          │
        ▼         ▼        ▼          ▼          ▼
    ┌────────┐ ┌────┐  ┌────────┐ ┌──────┐  ┌──────┐
    │ Dialer │ │SMS │  │ Email  │ │Review│  │Reject│
    │ System │ │Only│  │Platform│ │ Hold │  │ Log  │
    └────────┘ └────┘  └────────┘ └──────┘  └──────┘
    Call+SMS   SMS     Email-Only  Manual   Invalid
               Only    Email+SMS   Check    Leads
                      NurtureMail
             │
             ▼
    [Action 18: Notifications]
    Slack alert with summary
```

## Scoring Calculation Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         SCORING ARCHITECTURE                             │
└─────────────────────────────────────────────────────────────────────────┘

Input Data ──────────┬──────────────┬──────────────┬─────────────┐
                     │              │              │             │
                     ▼              ▼              ▼             ▼
              ┌──────────┐   ┌──────────┐   ┌──────────┐  ┌──────────┐
              │    DI    │   │    UI    │   │   OLS    │  │    CI    │
              │ Distress │   │ Urgency  │   │  Offer   │  │ Contact  │
              │  Index   │   │  Index   │   │Likelihood│  │  -able   │
              └─────┬────┘   └─────┬────┘   └─────┬────┘  └─────┬────┘
                    │              │              │             │
                    │   Weight:    │   Weight:    │  Weight:    │  Weight:
                    │     30%      │     25%      │    25%      │    20%
                    │              │              │             │
                    └──────┬───────┴──────┬───────┴─────┬───────┘
                           │              │             │
                           ▼              ▼             ▼
                        ┌────────────────────────────────┐
                        │    Overall Profitability      │
                        │         Score (OPS)            │
                        │          (0-1 scale)           │
                        └───────────────┬────────────────┘
                                        │
                        ┌───────────────┼───────────────┐
                        │               │               │
                        ▼               ▼               ▼
                   OPS ≥ 0.80      0.60-0.79       0.40-0.59
                   ┌─────────┐    ┌─────────┐    ┌─────────┐
                   │ TIER 1  │    │ TIER 2  │    │ TIER 3  │
                   │   HOT   │    │  WARM   │    │ NURTURE │
                   │ Leads   │    │ Leads   │    │ Leads   │
                   └─────────┘    └─────────┘    └─────────┘
```

## Distress Index (DI) Calculation Detail

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      DISTRESS INDEX (DI) LOGIC                           │
└─────────────────────────────────────────────────────────────────────────┘

Input Variables:
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ Estimated Equity │  │   Event Type     │  │Property Condition│
│   Percentage     │  │                  │  │   Score (1-5)    │
└────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘
         │                     │                     │
         ▼                     ▼                     ▼
    ┌─────────┐          ┌─────────┐          ┌─────────┐
    │Normalize│          │  Assign │          │Calculate│
    │Equity % │          │ Urgency │          │Condition│
    │÷ 50, max│          │  Score  │          │ Penalty │
    │  = 1.0  │          │         │          │         │
    └────┬────┘          └────┬────┘          └────┬────┘
         │                    │                    │
         │ Weight:            │ Weight:            │ Weight:
         │   35%              │   40%              │   25%
         │                    │                    │
         └────────────┬───────┴─────┬──────────────┘
                      │             │
                      ▼             ▼
              ┌────────────────────────┐
              │   DI = Σ(weighted)     │
              │   Range: 0.0 to 1.0    │
              └───────────┬────────────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
              ▼           ▼           ▼
         DI ≥ 0.70    0.40-0.69    DI < 0.40
         ┌──────┐    ┌──────┐     ┌──────┐
         │ HIGH │    │MEDIUM│     │ LOW  │
         │Tier  │    │Tier  │     │Tier  │
         └──────┘    └──────┘     └──────┘

Event Urgency Mapping:
• Auction/Foreclosure → 1.0 (max distress)
• Tax Lien → 0.85
• Divorce → 0.60
• Probate → 0.50
• Pre-Probate → 0.30
```

## Channel Routing Decision Tree

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CHANNEL ROUTING LOGIC TREE                            │
│                   (NO DIRECT MAIL OPTIONS)                               │
└─────────────────────────────────────────────────────────────────────────┘

                        [Lead Processed]
                              │
                              ▼
                    ┌──────────────────┐
                    │ Quality Check    │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
   Quality =           Quality =            Quality =
    Invalid           Incomplete            Clean or
        │                  │              DefaultsOnly
        ▼                  ▼                    │
    ┌──────┐          ┌──────┐                 │
    │REJECT│          │ HOLD │                 │
    └──────┘          └──────┘                 │
                                               ▼
                                    ┌───────────────────┐
                                    │ CI Score Check    │
                                    └─────────┬─────────┘
                                              │
                    ┌─────────────────────────┼─────────────────────┐
                    │                         │                     │
                    ▼                         ▼                     ▼
              CI ≥ 0.70                   CI ≥ 0.50            CI < 0.50
              Has Cell?                   Has Email?           Has Email?
                    │                         │                     │
          ┌─────────┴─────────┐      ┌────────┴────────┐           │
          ▼                   ▼      ▼                 ▼           ▼
        Yes                  No    Yes                No        ┌──────┐
          │                   │      │                 │        │ HOLD │
          ▼                   ▼      ▼                 ▼        └──────┘
    ┌──────────┐        ┌──────┐  ┌─────────┐    ┌──────┐
    │OPS Check │        │ HOLD │  │OPS Check│    │ HOLD │
    └────┬─────┘        └──────┘  └────┬────┘    └──────┘
         │                              │
    ┌────┴────┐                    ┌────┴────┐
    ▼         ▼                    ▼         ▼
OPS≥0.60  OPS<0.60            OPS≥0.60  OPS<0.60
    │         │                    │         │
    ▼         ▼                    ▼         ▼
┌────────┐ ┌───────┐         ┌────────┐ ┌──────────┐
│Call+SMS│ │SMS-Only│        │Email+  │ │Nurture   │
│        │ │        │         │SMS     │ │Mail      │
└────────┘ └───────┘         └────────┘ └──────────┘
(Dialable)  (No Dial)        (No Dial)  (No Dial)

Channel Outputs:
• Call+SMS: High-touch (calls + texts)
• SMS-Only: Medium-touch (texts only)
• Email+SMS: Multi-channel digital
• Email-Only: Email campaigns
• NurtureMail: Long-term email nurture
• HOLD: Manual review needed
• REJECT: Do not process
```

## Data Flow Through GPT-5 Pro

```
┌─────────────────────────────────────────────────────────────────────────┐
│               GPT-5 PRO PROCESSING ARCHITECTURE                          │
└─────────────────────────────────────────────────────────────────────────┘

Raw Lead Data ──┐
                │
                ▼
        ┌───────────────┐
        │ GPT-5 Pro API │  ← Temperature: 0.1 (deterministic)
        │   Endpoint    │  ← Response Format: JSON
        └───────┬───────┘  ← System Role: Data Specialist
                │
        ┌───────┴────────────────────┐
        │                            │
        ▼                            ▼
    [Prompt]                    [Context]
    • Task instructions         • Lead data
    • Formula to apply          • Field values
    • Rules to follow           • Current state
    • Output JSON schema        • Dependencies
        │                            │
        └────────────┬───────────────┘
                     │
                     ▼
            ┌────────────────┐
            │   GPT-5 Pro    │
            │   Processing   │
            │                │
            │ • Analyzes     │
            │ • Calculates   │
            │ • Classifies   │
            │ • Validates    │
            └───────┬────────┘
                    │
                    ▼
            ┌───────────────┐
            │ JSON Response │
            └───────┬───────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
        ▼           ▼           ▼
    [Score]    [Classification] [Validation]
    Numeric    Category/Tier    Issues List
    0-1 range  H/M/L or text    Array
        │           │           │
        └───────────┼───────────┘
                    │
                    ▼
          Next Workflow Action

Example Flow for DI Calculation:
───────────────────────────────
Input → GPT-5 → Output
{         │      {
equity:   │        "di_score": 0.7123,
  35%,    │        "equity_normalized": 0.70,
event:    │        "event_urgency": 1.0,
  Auction,│        "condition_penalty": 0.40,
cond: 3   │        "reasoning": "High urgency..."
}         │      }
```

## Processing Timeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   SINGLE LEAD PROCESSING TIME                            │
└─────────────────────────────────────────────────────────────────────────┘

CSV Upload ────────────► 1-2 seconds
    │
Table Storage ─────────► 1 second
    │
Extraction ────────────► 2 seconds
    │
GPT Cleaning (4 calls) ► 8-12 seconds
    │
GPT Scoring (5 calls) ─► 10-15 seconds
    │
GPT Routing (3 calls) ─► 6-9 seconds
    │
Compilation ───────────► 2 seconds
    │
Table Storage ─────────► 1 second
    │
Channel Routing ───────► 2-5 seconds
    │
Notification ──────────► 1 second
    │
TOTAL: ~35-50 seconds per lead

Batch Processing:
• 100 leads: ~60-90 minutes
• 500 leads: ~5-7 hours
• 1000 leads: ~10-14 hours

Optimization:
• Parallel processing possible for some GPT calls
• Can reduce to ~25-35 seconds per lead with optimization
```

## Integration Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  EXTERNAL SYSTEM INTEGRATIONS                            │
└─────────────────────────────────────────────────────────────────────────┘

                    ┌──────────────┐
                    │   Zapier     │
                    │   Platform   │
                    └──────┬───────┘
                           │
        ┏━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━┓
        ┃                                    ┃
        ▼                                    ▼
┌────────────────┐                  ┌────────────────┐
│   Input Side   │                  │  Output Side   │
└────────────────┘                  └────────────────┘
        │                                    │
    ┌───┴───┐                        ┌───────┴────────┬─────────┐
    │       │                        │                │         │
    ▼       ▼                        ▼                ▼         ▼
┌──────┐ ┌─────┐              ┌─────────┐    ┌──────────┐  ┌──────┐
│ CSV  │ │ UI  │              │ Dialer  │    │  Email   │  │ Slack│
│ File │ │Form │              │ Webhook │    │ Platform │  │ API  │
└──────┘ └─────┘              └─────────┘    └──────────┘  └──────┘
                              (Call+SMS)      (Mailchimp,  (Alerts)
                                              ActiveCamp,
                                              HubSpot)
                                      │                │
                                      ▼                ▼
                              ┌──────────┐    ┌──────────┐
                              │ Phone    │    │ Email    │
                              │ System   │    │ Campaign │
                              └──────────┘    └──────────┘

Additional Integration Points:
• CRM Systems (Salesforce, Podio, REsimpli)
• SMS Platforms (Twilio, EZ Texting)
• DNC Lookup Services
• Email Verification APIs (ZeroBounce, NeverBounce)
• Property Data Enrichment APIs
```

## Cost Structure Breakdown

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    COST ANALYSIS PER LEAD                                │
└─────────────────────────────────────────────────────────────────────────┘

Component           │ Tasks/Calls │ Unit Cost  │ Per Lead Cost
────────────────────┼─────────────┼────────────┼──────────────
Zapier Tasks        │     20      │  $0.006    │   $0.12
(Professional Plan) │             │            │
────────────────────┼─────────────┼────────────┼──────────────
GPT-5 Pro API       │     14      │  $0.002    │   $0.03
(avg per call)      │             │            │
────────────────────┼─────────────┼────────────┼──────────────
External APIs       │    2-3      │  $0.001    │   $0.003
(optional)          │             │            │
────────────────────┴─────────────┴────────────┴──────────────
TOTAL PER LEAD: ~$0.15-0.18

Volume Pricing:
┌────────────┬──────────┬──────────┬──────────┐
│   Volume   │  Zapier  │ OpenAI   │  Total   │
├────────────┼──────────┼──────────┼──────────┤
│ 100 leads  │   $12    │   $3     │   $15    │
├────────────┼──────────┼──────────┼──────────┤
│ 500 leads  │   $60    │   $15    │   $75    │
├────────────┼──────────┼──────────┼──────────┤
│ 1000 leads │   $120   │   $30    │   $150   │
├────────────┼──────────┼──────────┼──────────┤
│ 5000 leads │   $600   │   $150   │   $750   │
└────────────┴──────────┴──────────┴──────────┘

Note: Plus Zapier subscription ($49-299/mo base)
```

---

## Legend

```
Shapes & Symbols:
┌────┐   Standard process box
│    │
└────┘

┌────┐   Decision point
│ ?? │
└────┘

[Box]    GPT-5 Pro action

{Box}    Data storage

╔════╗   Section header
║    ║
╚════╝

──►      Data flow direction
═══►     Major process flow

Tiers:
H = High
M = Medium  
L = Low
```

This diagram set provides visual representation of the complete workflow for stakeholders and implementation teams.
