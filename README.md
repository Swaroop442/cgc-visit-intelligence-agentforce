# Salesforce Visit Intelligence Agent (Agentforce)

A next-generation Agentforce Employee Agent designed to assist internal supervisors and field representatives in managing Consumer Goods Cloud (CGC) Visit records and associated Account context.

This agent runs on the Salesforce **Atlas Reasoning Engine** and uses advanced natural language understanding to automate visit searching, aggregate metrics, analyze historical purchasing trends, and update delivery notes on-the-fly.

---

## 🚀 Key Features

- **Page-Context Awareness**: Automatically initializes when loaded inside a Visit record page context, loading associated store details and greetings without any user configuration.
- **Natural Language Visit Search**: Dynamic parsing of natural language query search phrases (e.g., `"visits tomorrow"`, `"this week"`, or specific store names) to locate upcoming or historical visits.
- **Account Health Summarizer**: Aggregates counts of related Contacts, pending Tasks, active Promotions, Tactics, Assortments, Store Products, and Out-of-Stock issues in a single inquiry.
- **Purchasing & Order Summary Insights**: Pulls historical order spend data, categorizes orders by phase (Draft, Submitted, Canceled), aggregates brand/product frequency stats, and exposes the top-selling product.
- **Secure Database Writebacks**: Updates order delivery instructions/notes, protected by Agentforce's built-in User Confirmation gates.
- **Fallback Mock Data**: Built-in mock response fallbacks inside backing Apex classes to ensure smooth and uninterrupted Agent previews in empty sandbox/developer environments.

---

## 🛠️ Repository File Structure

To showcase or deploy this agent, make sure to commit/upload the following files from your Salesforce DX project:

```plaintext
├── force-app/main/default/
│   ├── aiAuthoringBundles/
│   │   └── Visit_Intelligence/
│   │       ├── Visit_Intelligence.agent            # Core Agent Script (instructions, FSM, and variables)
│   │       └── Visit_Intelligence.bundle-meta.xml  # Agent bundle metadata XML
│   ├── classes/
│   │   ├── GetAvailableVisits.cls                  # Apex: Queries today's and upcoming visits
│   │   ├── GetAvailableVisitsTest.cls              # Apex Test for GetAvailableVisits
│   │   ├── SearchVisitsForAgent.cls                # Apex: NL date parsing and keyword visit query
│   │   ├── SearchVisitsForAgentTest.cls            # Apex Test for SearchVisitsForAgent
│   │   ├── GetAccountRelatedCounts.cls             # Apex: Gathers record counts for account dashboard
│   │   ├── GetAccountRelatedCountsTest.cls         # Apex Test for GetAccountRelatedCounts
│   │   ├── GetAccountRelatedRecords.cls            # Apex: Fetches detailed listings by category
│   │   ├── GetAccountRelatedRecordsTest.cls        # Apex Test for GetAccountRelatedRecords
│   │   ├── GetAccountOrderSummaryForAgent.cls      # Apex: Spend metrics & ordering insight compiler
│   │   ├── GetOrderLineItems.cls                   # Apex: Queries product details inside orders
│   │   ├── GetOrderLineItemsTest.cls               # Apex Test for GetOrderLineItems
│   │   ├── UpdateOrderDeliveryNote.cls             # Apex: Performs generic database DML updates
│   │   └── UpdateOrderDeliveryNoteTest.cls         # Apex Test for UpdateOrderDeliveryNote
│   └── flows/
│       └── Get_Visit_Details.flow-meta.xml         # Autolaunched Flow: Resolves visit context records
└── sfdx-project.json                               # Salesforce DX project file
```

---

## 📊 Conversation State Machine

The agent routes conversations between modular states using the **Hub-and-Spoke** architecture pattern:

Here is the link to view the Flow Diagram :
https://excalidraw.com/#json=ng05HfV08Vg90Ca8ABED3,a3VsxXFFEOVPiotO0xarwQ

---

## ⚙️ Deployment & Activation Pipeline

Follow these sequential steps to compile, deploy, and activate the agent in a target Salesforce Org.

### Step 1: Pre-Deployment Prerequisites

1. **Turn on Einstein**: Go to **Einstein Setup** in Setup and enable **Einstein**.
2. **Enable Agentforce**: Go to **Agentforce Settings** in Setup and enable **Agentforce Agents**.

### Step 2: Deploy Backing Logic & Apex Tests

Deploy all Apex classes, tests, and Flows first. The agent bundle cannot compile if its target endpoints do not exist in the org.

# Deploy Flow

sf project deploy start --metadata Flow:Get_Visit_Details

# Deploy Apex Classes

sf project deploy start --metadata ApexClass:GetAvailableVisits ApexClass:SearchVisitsForAgent ApexClass:GetAccountRelatedCounts ApexClass:GetAccountRelatedRecords ApexClass:GetAccountOrderSummaryForAgent ApexClass:GetOrderLineItems ApexClass:UpdateOrderDeliveryNote

# Run tests to guarantee unit test coverage (minimum 75% required for production)

sf project test run --tests GetAvailableVisitsTest SearchVisitsForAgentTest GetAccountRelatedCountsTest GetAccountRelatedRecordsTest GetOrderLineItemsTest UpdateOrderDeliveryNoteTest --result-format human

### Step 3: Validate and Deploy the Agent Bundle
Once backing components are active, validate that your agent compiles cleanly on the platform:
```bash
# Validate local agent bundle compilation
sf agent validate authoring-bundle --api-name Visit_Intelligence

# Deploy Agent authoring bundle metadata
sf project deploy start --metadata GenAiPlannerBundle:Visit_Intelligence
````

### Step 4: Publish and Activate

Publishing the bundle freezes your script into an immutable version.

# Publish version

sf agent publish authoring-bundle --api-name Visit_Intelligence

# Activate the published version to expose it to users

sf agent activate --api-name Visit_Intelligence

### Step 5: Post-Deployment Security Permissions

After activating, ensure your users' Profiles and Permission Sets are configured to allow access to the agent and classes:

1. **Apex Class Access**: Go to the target user **Profiles** (or a custom Permission Set) $\rightarrow$ **Enabled Apex Class Access** $\rightarrow$ add all backing classes:
   - `GetAvailableVisits`
   - `SearchVisitsForAgent`
   - `GetAccountRelatedCounts`
   - `GetAccountRelatedRecords`
   - `GetAccountOrderSummaryForAgent`
   - `GetOrderLineItems`
   - `UpdateOrderDeliveryNote`
2. **Agent Access Permissions**:
   - Ensure users have the **Einstein Agent User** permission set assigned.
   - For Employee agents, configure the **Agent Access** settings to define which profiles or permission set groups can see the agent utility inside the Salesforce UI.

---

## 🔒 Safety & Guardrails

- **Off-Topic Redirection**: Specialized instructions in the `off_topic` and `ambiguous_question` subagents block general knowledge queries, preventing hallucination or misuse.
- **DML Protection**: Any data modification (e.g. updating delivery instructions via `UpdateOrderDeliveryNote`) enforces `require_user_confirmation: True`, requiring the user to explicitly click "Confirm" in the UI before writing to the database.
