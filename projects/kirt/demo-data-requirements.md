# KIRT — Demo Data Requirements

> Derived from `kirt-spec-seed.md`. Everything KIRT V1 needs to demonstrate, worked backwards to the data that must exist. Covers real data extraction, gap identification, and full synthetic generation.

---

## 1. What You Need (Human-Readable Summary)

### Accounts (4+ required)

Each demo account needs a complete data package. The matrix:

| Account | Type | Purpose |
|---------|------|---------|
| Account A | Public company, existing client | Full data richness — SEC filings, earnings, engagement history, SF pipeline |
| Account B | Public company, new prospect | Greenfield research — shows KIRT's value with public data and zero internal history |
| Account C | Private company, existing client | Rich internal data, limited public data — tests how KIRT performs without SEC/earnings |
| Account D | Private company, new prospect | Thin data — stress-tests the platform on minimal inputs |

### Per-Account Data Package

For each account, you need:

**Salesforce data (CSV exports):**
- Account record (name, industry, revenue, employees, website, description, owner, tier)
- Contacts (3-8 per account: name, title, department, email, phone, last activity)
- Opportunities (2-5 per account: name, stage, amount, close date, probability, next step, competitor)
- Activities/tasks (5-10 per account: subject, description, date, status, related contact)
- Notes (2-4 per account: title, body, related record, created date)

**SharePoint documents (per account folder):**
- 1 BSE or account profile document (.docx)
- 1 SOW or engagement scope document (.docx)
- 1-2 meeting notes (.docx) — varying quality levels
- 1 deck/presentation (.pptx) — pitch, assessment, or strategic review
- 1 proposal or deliverable from a past engagement (.docx or .pdf)

**Public data (for public company accounts only):**
- Gathered at runtime via AI research workflow — not pre-staged files
- KIRT's research pipeline pulls SEC filings, earnings data, news, and company profiles on demand
- For demo, the accounts selected must be real public companies (or realistic synthetic ones) so live research can execute against actual public sources
- No pre-built public data directory needed in the demo data package

**For the dedup demo:**
- 1 document that exists in 3+ locations with identical content (different filenames)
- 1 document that exists as both .docx and .pdf (cross-format)

### Reference Data (not per-account)

**Offerings catalog (v0.1):**
- Solution domains (5-8 domains: Cloud Engineering, Data & Analytics, AI, Cyber Security, Managed Services, Managed Security, etc.)
- Offerings per domain (3-6 each, with name, description, maturity rating: ready/emerging/sunset)
- Total: 20-40 offerings across all domains

**Discovery question taxonomy:**
- Tree structure: business → industry → solution modules
- Full question set (all 200+ questions) — this is a working demo, not a reduced prototype
- Each question tagged with: module, applicable industries, related offerings

**Deliverable templates:**
- Account Brief template (.docx) — section headers matching the spec (overview, contacts, priorities, history, opportunities, competitive, strategic notes, white space, action items)
- Engagement Prep template (.docx) — sections for prior work, pain points, stakeholder map, talking points, discovery questions
- Client Deliverable template (.docx) — generic client-ready format
- White Space Grid template (.xlsx or .json) — matrix of solution domains/offerings (rows) vs. account engagement status (columns: active, past, identified opportunity, no engagement). Populated per account using the offerings catalog as the row axis and SF/SP engagement data as the column values. This is a core Phase 1 output that feeds cross-sell recommendations and the Account Brief's white space section.
- Each template should have placeholder text showing expected content per section

**Stakeholder map seed data (per existing client account):**
- 3-5 key contacts with roles (decision maker, champion, blocker, influencer, technical lead)
- Reporting relationships where known

### Admin Configuration Data

**Location-ranking config:**
- Ordered list of SP locations by authority (e.g., `/Official-Library/` > `/Team-Sites/` > `/Personal/`)

**Scoring weights:**
- Growth scoring: 5 dimensions, equal starting weights (20% each)
- Tier thresholds: score ranges for Tier 1 / Tier 2 / Tier 3

**LLM provider config:**
- Default provider and model for generation tasks
- Fallback provider

---

## 2. Validating Real Data & Identifying Gaps

Use this section when working with extracted real data. For each item, validate what was provided, flag what's missing, and determine whether to fill gaps with synthetic data or defer.

### Account-Level Validation

For each account in the dataset, check:

```
Account: _______________

SALESFORCE DATA
[ ] Account record exists with: name, industry, revenue, employees, website
    Missing fields: _______________
[ ] 3+ contacts with name, title, department, email
    Count provided: ___  Missing fields: _______________
[ ] 2+ opportunities with stage, amount, close date
    Count provided: ___  Missing fields: _______________
[ ] 5+ activities/tasks with dates
    Count provided: ___  Missing fields: _______________
[ ] Notes attached to account/contacts/opportunities
    Count provided: ___

SHAREPOINT DOCUMENTS
[ ] BSE or account profile (.docx)
    Status: exists / missing / partial
[ ] SOW or engagement scope (.docx)
    Status: exists / missing / partial
[ ] Meeting notes (at least 1, ideally 2+)
    Count provided: ___  Quality: clean / mixed / messy
[ ] Deck or presentation (.pptx)
    Status: exists / missing / partial
[ ] Past deliverable (.docx or .pdf)
    Status: exists / missing / partial

PUBLIC DATA (public companies only)
    N/A — gathered at runtime via AI research workflow
    [ ] Account is a real public company (or realistic synthetic) so live research can execute
    [ ] Research pipeline has API access to public sources (SEC EDGAR, news APIs)

DEDUP TEST DATA
[ ] At least 1 doc in 3+ locations (identical content, different names)
[ ] At least 1 doc as .docx + .pdf pair

COVERAGE ASSESSMENT
[ ] Sufficient for Account Brief generation?      yes / no / partial
[ ] Sufficient for Engagement Prep generation?     yes / no / partial
[ ] Sufficient for Cross-sell recommendation?      yes / no / partial
[ ] Sufficient for search demo (keyword + semantic)? yes / no / partial
```

### Reference Data Validation

```
OFFERINGS CATALOG
[ ] Solution domains defined (target: 5-8)
    Count provided: ___
[ ] Offerings per domain (target: 3-6 each)
    Total offerings: ___
[ ] Maturity ratings assigned (ready / emerging / sunset)
    Rated: ___ / ___  Unrated: ___

DISCOVERY QUESTIONS
[ ] Full question set available (target: all 200+)
    Count provided: ___
[ ] Structured as tree (business / industry / solution)?
    Structure: tree / flat / unstructured notes
[ ] Tagged with module and applicable industries?
    Tagged: ___ / ___  Untagged: ___

DELIVERABLE TEMPLATES
[ ] Account Brief template with section headers
[ ] Engagement Prep template with section headers
[ ] Client Deliverable template
[ ] White Space Grid template (offerings x engagement status matrix)
```

### Gap Decision Matrix

For each gap found, decide:

| Gap | Impact on Demo | Action |
|-----|---------------|--------|
| Missing SF field (e.g., no revenue) | Low — generation still works, brief has a blank | Synthesize the field |
| Missing entire SF object (e.g., no opportunities) | High — cross-sell and white space break | Synthesize full object or pick different account |
| No SP documents for an account | High — search returns nothing, brief is thin | Synthesize 3-5 docs or pick different account |
| No public data (private company) | Expected — this IS the demo scenario | Leave as-is, demo shows thin-data behavior |
| Public data research pipeline not working | High — public company accounts lose key differentiator | Fix pipeline; public data is gathered at runtime, not pre-staged |
| No offerings catalog | Critical — white space, scoring, cross-sell all break | Must create v0.1 before demo |
| No discovery questions | High — engagement prep loses key feature | Must extract or create full 200+ set |
| No white space grid template | High — Phase 1 core output missing | Must create template |
| No templates | High — generation has no format to follow | Must create templates |

---

## 3. Complete Synthetic Data Generation

Use this section to generate a full demo dataset from scratch. Produces everything needed for a working V1 demo without any real data.

### Directory Structure

```
demo-data/
├── config/
│   ├── offerings-catalog.json
│   ├── discovery-questions.json
│   ├── location-ranking.json
│   └── scoring-weights.json
├── templates/
│   ├── Account-Brief-Template.docx
│   ├── Engagement-Prep-Template.docx
│   ├── Client-Deliverable-Template.docx
│   └── White-Space-Grid-Template.xlsx
├── salesforce/
│   ├── accounts.csv
│   ├── contacts.csv
│   ├── opportunities.csv
│   ├── activities.csv
│   └── notes.csv
├── sharepoint/
│   ├── Apex-Industries/
│   │   ├── Apex-Industries-Account-Profile-2025.docx
│   │   ├── Apex-Industries-SOW-Cloud-Migration-2025.docx
│   │   ├── Apex-Industries-QBR-Meeting-Notes-2026-01-15.docx
│   │   ├── Apex-Industries-QBR-Meeting-Notes-2026-02-20.docx
│   │   ├── Apex-Industries-Security-Assessment-Deck.pptx
│   │   └── Apex-Industries-Cloud-Strategy-Proposal-2025.docx
│   ├── Meridian-Health/
│   │   ├── Meridian-Health-BSE-2025.docx
│   │   ├── Meridian-Health-SOW-SOC-Services-2024.docx
│   │   ├── Meridian-Health-Discovery-Meeting-Notes-2026-03-01.docx
│   │   └── Meridian-Health-Managed-Security-Pitch.pptx
│   ├── Cobalt-Financial/
│   │   ├── Cobalt-Financial-Account-Profile-2025.docx
│   │   ├── Cobalt-Financial-Data-Analytics-SOW-2025.docx
│   │   ├── Cobalt-Financial-Executive-Briefing-Notes-2026-02-10.docx
│   │   ├── Cobalt-Financial-Meeting-Notes-2026-03-05.docx
│   │   └── Cobalt-Financial-AI-Readiness-Assessment.pptx
│   ├── Terraform-Logistics/
│   │   ├── Terraform-Logistics-Prospect-Research-2026.docx
│   │   └── Terraform-Logistics-Initial-Meeting-Notes-2026-03-10.docx
│   └── _dedup-test/
│       ├── Official-Library/
│       │   └── Apex-Industries-Cloud-Strategy-Proposal-2025.docx
│       ├── Team-Site/
│       │   └── Apex-Cloud-Proposal-FINAL.docx
│       ├── Personal/
│       │   └── apex cloud strategy.docx
│       └── Cross-Format/
│           ├── Cobalt-Financial-AI-Readiness-Assessment.docx
│           └── Cobalt-Financial-AI-Readiness-Assessment.pdf
└── stakeholder-maps/
    ├── Apex-Industries-Stakeholders.json
    ├── Meridian-Health-Stakeholders.json
    └── Cobalt-Financial-Stakeholders.json
```

### Synthetic Account Profiles

| Account | Type | Industry | Revenue | Employees | Scenario |
|---------|------|----------|---------|-----------|----------|
| Apex Industries | Public, existing client | Manufacturing / Industrial | $4.2B | 12,000 | Full data richness. Cloud migration engagement in progress, security assessment completed, strong relationship. |
| Cobalt Financial Group | Public, new prospect | Financial Services | $8.7B | 28,000 | Greenfield. Strong public data (SEC, earnings). No internal engagement history. Exploring data analytics and AI. |
| Meridian Health Systems | Private, existing client | Healthcare | $650M | 3,200 | Active managed security client. Limited public data. SOC services SOW, multiple meeting notes. Tests MSA sensitivity. |
| Terraform Logistics | Private, new prospect | Transportation / Logistics | $180M | 800 | Thin data. One initial meeting note. No SF history, no public filings. Stress-tests KIRT on minimal input. |

### Salesforce CSV Schemas

**accounts.csv**
```
sf_record_id,name,industry,annual_revenue,employee_count,website,description,billing_city,billing_state,account_owner,tier,created_date
```

**contacts.csv**
```
sf_record_id,first_name,last_name,title,department,email,phone,account_name,account_id,lead_source,last_activity_date
```

**opportunities.csv**
```
sf_record_id,name,account_name,account_id,stage,amount,close_date,probability,next_step,description,competitor,opportunity_owner,created_date
```

**activities.csv**
```
sf_record_id,subject,description,activity_date,status,type,related_account,related_contact,assigned_to
```

**notes.csv**
```
sf_record_id,title,body,related_record_name,related_record_type,created_date,created_by
```

### Config File Schemas

**offerings-catalog.json**
```json
{
  "version": "0.1",
  "generated": "2026-03-24",
  "domains": [
    {
      "name": "Cloud Engineering",
      "offerings": [
        {
          "name": "Cloud Migration & Modernization",
          "description": "...",
          "maturity": "ready",
          "discovery_question_tags": ["cloud", "migration", "infrastructure"]
        }
      ]
    }
  ]
}
```

**discovery-questions.json**
```json
{
  "version": "0.1",
  "modules": {
    "business": [
      {
        "id": "BIZ-001",
        "question": "What are your top 3 business priorities for the next 12 months?",
        "industries": ["all"],
        "related_offerings": []
      }
    ],
    "industry": [
      {
        "id": "IND-FS-001",
        "question": "How are you addressing regulatory requirements around data residency and AI governance?",
        "industries": ["financial_services"],
        "related_offerings": ["data-governance", "compliance-automation"]
      }
    ],
    "solution": [
      {
        "id": "SOL-CLOUD-001",
        "question": "What percentage of your workloads are currently in public cloud vs. on-premises?",
        "industries": ["all"],
        "related_offerings": ["cloud-migration", "hybrid-infrastructure"]
      }
    ]
  }
}
```

**location-ranking.json**
```json
{
  "rankings": [
    {"pattern": "*/Official-Library/*", "rank": 1},
    {"pattern": "*/PMO/*", "rank": 2},
    {"pattern": "*/Team-Sites/*", "rank": 3},
    {"pattern": "*/KIRT-Generated/*", "rank": 4},
    {"pattern": "*/Personal/*", "rank": 5}
  ]
}
```

**scoring-weights.json**
```json
{
  "growth_scoring": {
    "version": "0.1",
    "dimensions": {
      "revenue": 0.20,
      "strategic_value": 0.20,
      "relationship_depth": 0.20,
      "engagement_frequency": 0.20,
      "white_space_size": 0.20
    },
    "tier_thresholds": {
      "tier_1": 80,
      "tier_2": 50,
      "tier_3": 0
    }
  }
}
```

**stakeholder map (per account)**
```json
{
  "account": "Apex Industries",
  "stakeholders": [
    {
      "name": "Sarah Chen",
      "title": "CTO",
      "role": "decision_maker",
      "relationship_status": "strong",
      "notes": "Champion for cloud modernization. Reports to CEO.",
      "reports_to": "Michael Torres"
    }
  ]
}
```

### Document Content Guidelines

**Meeting notes** should include:
- Date, attendees, location
- 3-5 discussion topics as prose paragraphs (not bullet lists — tests NLP extraction)
- At least 1 action item buried in text
- At least 1 competitive mention or pain point for signal extraction
- Varying quality: Account A gets polished notes, Account D gets raw/informal

**BSE / account profiles** should include:
- Company background paragraph
- Current technology landscape (2-3 paragraphs)
- Known pain points and strategic initiatives
- Prior engagement history
- Key relationships

**SOW documents** should include:
- Engagement scope and objectives
- Deliverables list
- Timeline
- Pricing section (anonymized)
- References to specific offerings from the catalog

**Decks (.pptx)** should include:
- Title slide with account name and date
- 5-10 content slides covering assessment findings or strategic recommendations
- At least 1 slide with a table (tests classification and extraction)

**White Space Grid** should include:
- Rows: every offering from the offerings catalog, grouped by solution domain
- Columns: engagement status per account (active engagement, past engagement, identified opportunity, no engagement)
- Template should be empty/placeholder — KIRT populates it per account by cross-referencing the catalog against SF opportunities and SP engagement documents
- Output format: structured data (JSON or Excel) that KIRT renders as a visual grid in the UI

**Public data:**
- Not pre-staged. Gathered at runtime via KIRT's AI research pipeline.
- For synthetic accounts that are fictional companies, the research pipeline should gracefully handle "no public data found" as a valid state (this is the private company / new prospect scenario).

### Dedup Test Data

The `_dedup-test/` directory contains:
1. **Exact duplicates (3 copies):** Same Apex proposal in three locations with different filenames. Content is byte-identical. Tests content-hash dedup and location-ranking config.
2. **Cross-format pair:** Same Cobalt assessment as .docx and .pdf. Tests cross-format awareness (V1 shows both, V2 dedup handles it).

**White Space Grid template schema:**
```json
{
  "account": "Apex Industries",
  "generated": "2026-03-24",
  "grid": [
    {
      "domain": "Cloud Engineering",
      "offerings": [
        {
          "offering": "Cloud Migration & Modernization",
          "status": "active",
          "evidence": ["SOW-Cloud-Migration-2025", "OPP-001"],
          "notes": "Phase 2 in progress, $1.2M engagement"
        },
        {
          "offering": "Hybrid Infrastructure Management",
          "status": "identified_opportunity",
          "evidence": ["Meeting-Notes-2026-02-20"],
          "notes": "CTO mentioned hybrid pain points in Q1 QBR"
        }
      ]
    },
    {
      "domain": "Cyber Security",
      "offerings": [
        {
          "offering": "Security Assessment",
          "status": "past",
          "evidence": ["Security-Assessment-Deck.pptx"],
          "notes": "Completed 2025, no follow-on yet"
        },
        {
          "offering": "SOC Services",
          "status": "no_engagement",
          "evidence": [],
          "notes": ""
        }
      ]
    }
  ]
}
```

### Minimum Viable Dataset

This is a working demo — all data should be complete and production-representative.

**Priority 1 — must exist before any feature works:**
1. **Config files** (offerings catalog with full domain/offering set, full 200+ discovery questions, scoring weights, location ranking)
2. **Templates** (Account Brief, Engagement Prep, Client Deliverable, White Space Grid)

**Priority 2 — demonstrates the core value proposition:**
3. **Account A (Apex Industries) full package** — SF data + SP docs + stakeholder map — demonstrates full-richness scenario. Public data gathered at runtime.
4. **Account D (Terraform Logistics) minimal package** — 1 meeting note + bare SF account record — demonstrates thin-data behavior

**Priority 3 — completes the demo:**
5. **Dedup test data** — 3 copies of one doc + 1 cross-format pair
6. **Remaining accounts** (B and C) — fill out the matrix
7. **AI research pipeline** operational for live public data gathering on public company accounts
