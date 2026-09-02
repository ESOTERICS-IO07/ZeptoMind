ZEPTOMIND — MASTER ENGINEERING CONTRACT
Version: 1.0
Status: ARCHITECTURE LOCK / HACKATHON FREEZE
Build Window: 24 hours
Primary Goal: Build a working Agentic AI Financial Operating System for Quick Commerce.

This document is the single source of truth for architecture, interfaces, ownership, data structures, agent behavior, API contracts, integration rules, demo behavior, and deployment assumptions.

Rule: Internal implementation may change. Public contracts may not change without team agreement.

0. EXECUTIVE CONTRACT
0.1 Product
ZeptoMind is an agentic financial operating system that identifies operational leakage in quick-commerce dark stores, quantifies the leakage in monetary terms, simulates possible interventions, ranks actions, and routes the selected action through human approval before mocked execution.

0.2 Core business question
Where is the store losing money, what operational intervention can recover it, and what is the expected contribution improvement?

0.3 Primary metric
Contribution Margin / Order
Contribution / Order =
Revenue / Order
+ Product Margin / Order
- Discount / Order
- Delivery Cost / Order
- Fulfillment Cost / Order
- Dark Store Cost / Order
- Payment / Platform Cost / Order
- CAC Allocation / Order
0.4 Core closed loop
Operational Data
      ↓
Specialized Agents
      ↓
Unit Economics Engine
      ↓
CFO Agent / LangGraph
      ↓
Candidate Actions
      ↓
Simulation
      ↓
Ranked Recommendation
      ↓
Human Approval
      ↓
Mock Execution
      ↓
Recalculate Economics
      ↓
Measured Result
0.5 Non-goals
The 24-hour MVP will NOT implement:

Real Zepto production integration

Real financial transactions

Real credit disbursement

Real discount execution

Real rider dispatch

Full production routing/VRP

Kafka

RabbitMQ

Redis

WebSockets

Kubernetes

Fine-tuning an LLM

A large multi-agent swarm

A production-grade RAG system unless time remains after the core loop works

1. LOCKED TECH STACK
1.1 Frontend
React

TypeScript

Vite

Tailwind CSS

shadcn/ui

Recharts

REST API client using fetch or equivalent

Responsibilities:

Dashboard

Store selection

Financial metrics

Agent findings

CFO recommendation

Simulation visualization

Approval/rejection

Execution result

Browser-side temporary state/cache

1.2 Backend
Python

FastAPI

Pydantic

SQLAlchemy

AsyncIO

HTTPX

PostgreSQL

REST API

Deployment:

Render

Rules:

FastAPI endpoints should be async def where I/O is involved.

External HTTP calls use httpx.AsyncClient.

Database I/O should use async-compatible SQLAlchemy patterns.

Backend owns business orchestration.

Backend does not duplicate frontend calculations.

1.3 Database
PostgreSQL.

Recommended hosted providers:

Supabase

Neon

Database is the source of truth for persistent application state.

1.4 Agent orchestration
LangGraph

The graph is the orchestration/state machine layer.

The LLM is NOT the numerical calculation engine.

1.5 LLM
Claude or GPT API.

LLM responsibilities:

CFO reasoning

Explanation

Prioritization

Comparing agent findings

Generating human-readable rationale

LLM must NOT invent numerical outputs.

Numerical values must come from deterministic code, ML models, or simulation outputs.

1.6 ML
Python

Pandas

NumPy

scikit-learn

XGBoost

Models:

Inventory demand forecast

Overstock/stockout risk

Discount elasticity/effectiveness

Fulfillment capacity indicators

1.7 Simulation
Preferred:

NumPy Monte Carlo

Lightweight deterministic what-if simulation

Optional:

SimPy

1.8 Optimization
Preferred:

Simple heuristics

OR-Tools only where useful

1.9 Communication
Frontend ↔ Backend = REST
Backend ↔ External APIs = HTTPX
Backend ↔ Internal modules = Python function calls
No WebSockets.

No Redis event bus.

No Kafka.

2. REPOSITORY CONTRACT
2.1 Repository
zeptomind/
2.2 Required structure
zeptomind/
│
├── frontend/
│
├── backend/
│   └── app/
│       ├── api/
│       ├── db/
│       ├── schemas/
│       ├── services/
│       └── main.py
│
├── agents/
│   ├── inventory/
│   ├── discount/
│   └── fulfillment/
│
├── cfo/
│   ├── graph.py
│   ├── state.py
│   ├── nodes.py
│   ├── synthesizer.py
│   └── prompts/
│
├── simulation/
│
├── economics/
│
├── data/
│
├── docs/
│
├── contracts/
│
├── CONTRACTS.md
├── README.md
├── .env.example
├── .gitignore
└── docker-compose.yml
2.3 Ownership
Member	Role	Primary directories
P1	Frontend/Product	frontend/
P2	Backend/Platform	backend/
P3	Inventory + Discount ML	agents/inventory/, agents/discount/
P4	Fulfillment + Simulation	agents/fulfillment/, simulation/, economics/
P5	CFO + Tech Lead + Integration	cfo/, data/, contracts/, integration
3. GIT CONTRACT
3.1 Branches
main
│
└── integration
    │
    ├── feature/frontend
    ├── feature/backend
    ├── feature/ml-agents
    ├── feature/fulfillment-simulation
    └── feature/cfo-orchestration
3.2 Rules
Never directly push feature work to main.

Work on the assigned feature branch.

Push the feature branch.

Open a Pull Request into integration.

P5 performs integration review.

Integration branch is tested.

Stable integration is merged into main.

Never rewrite another member's branch.

Never commit secrets.

Never change a public contract silently.

3.3 Commit convention
Recommended:

feat:
fix:
refactor:
chore:
docs:
test:
Examples:

feat: add discount elasticity inference
feat: add store analysis endpoint
feat: add CFO decision graph
fix: handle empty inventory response
chore: initialize frontend
4. API CONTRACT
Base URL:

/api
4.1 GET /api/health
Response:

{
  "status": "ok"
}
4.2 GET /api/stores
Returns available stores.

Response:

{
  "stores": [
    {
      "id": "S142",
      "store_code": "S142",
      "city": "Bengaluru",
      "status": "ACTIVE"
    }
  ]
}
4.3 GET /api/stores/{store_id}
Returns store summary.

4.4 POST /api/analyze
Request:

{
  "store_id": "S142"
}
Response:

{
  "analysis_id": "AN001",
  "store_id": "S142",
  "status": "COMPLETED",
  "findings": [],
  "recommendations": []
}
This endpoint is the main entry point for the demo.

4.5 POST /api/simulate
Request:

{
  "store_id": "S142",
  "recommendation_id": "REC001"
}
Response:

{
  "scenario_id": "SIM001",
  "baseline": {},
  "projected": {},
  "impact": {}
}
4.6 GET /api/recommendations/{recommendation_id}
Returns full recommendation.

4.7 POST /api/recommendations/{recommendation_id}/approve
Request:

{
  "approved_by": "demo_manager"
}
4.8 POST /api/recommendations/{recommendation_id}/reject
Request:

{
  "rejected_by": "demo_manager",
  "reason": "Manual rejection"
}
4.9 POST /api/recommendations/{recommendation_id}/execute
Executes the mocked intervention.

No real-world operational action is performed.

5. HTTP / ERROR CONTRACT
Standard successful responses use HTTP 2xx.

Suggested errors:

{
  "error": {
    "code": "STORE_NOT_FOUND",
    "message": "Store S142 was not found."
  }
}
Required error codes:

STORE_NOT_FOUND
RECOMMENDATION_NOT_FOUND
SIMULATION_FAILED
AGENT_FAILED
INVALID_REQUEST
ALREADY_APPROVED
ALREADY_EXECUTED
INTERNAL_ERROR
Rules:

Do not expose API keys.

Do not expose stack traces in production responses.

Log internal exceptions on the backend.

Frontend displays a concise user-safe message.

6. DATABASE CONTRACT
6.1 stores
id                  UUID PK
store_code          VARCHAR UNIQUE NOT NULL
city                VARCHAR NOT NULL
orders_per_day      INTEGER
avg_order_value     NUMERIC(12,2)
status              VARCHAR
created_at          TIMESTAMP
updated_at          TIMESTAMP
6.2 orders
id                  UUID PK
store_id            UUID FK → stores.id
customer_id         VARCHAR
order_time          TIMESTAMP
order_value         NUMERIC(12,2)
product_margin      NUMERIC(12,2)
discount_amount     NUMERIC(12,2)
delivery_cost       NUMERIC(12,2)
fulfillment_cost    NUMERIC(12,2)
status              VARCHAR
6.3 inventory
id                  UUID PK
store_id            UUID FK → stores.id
sku_id              VARCHAR
current_stock       INTEGER
daily_demand        NUMERIC(12,2)
forecast_demand     NUMERIC(12,2)
unit_cost           NUMERIC(12,2)
expiry_days         INTEGER
stockout_risk       NUMERIC(5,4)
overstock_risk      NUMERIC(5,4)
6.4 discounts
id                      UUID PK
store_id                UUID FK → stores.id
customer_segment        VARCHAR
discount_amount         NUMERIC(12,2)
orders_without_discount INTEGER
orders_with_discount    INTEGER
incremental_orders      NUMERIC(12,4)
roi                     NUMERIC(12,4)
6.5 riders
id                  UUID PK
store_id            UUID FK → stores.id
available           BOOLEAN
orders_completed    INTEGER
avg_delivery_time   NUMERIC(12,2)
utilization         NUMERIC(5,4)
6.6 agent_findings
id                    UUID PK
store_id              UUID FK
agent_type            VARCHAR
finding_type          VARCHAR
description           TEXT
severity              VARCHAR
financial_impact      NUMERIC(14,2)
confidence            NUMERIC(5,4)
recommendation_data   JSONB
created_at            TIMESTAMP
6.7 recommendations
id                    UUID PK
store_id              UUID FK
problem               TEXT
action_type           VARCHAR
action_parameters     JSONB
impact_per_order      NUMERIC(14,2)
daily_impact          NUMERIC(14,2)
monthly_impact        NUMERIC(14,2)
confidence             NUMERIC(5,4)
risk                  VARCHAR
reasoning             TEXT
status                VARCHAR
created_at            TIMESTAMP
updated_at            TIMESTAMP
Allowed status:

PENDING
SIMULATED
APPROVED
REJECTED
EXECUTED
6.8 simulations
id                       UUID PK
recommendation_id        UUID FK
baseline_contribution    NUMERIC(14,2)
projected_contribution   NUMERIC(14,2)
improvement              NUMERIC(14,2)
confidence               NUMERIC(5,4)
parameters               JSONB
created_at               TIMESTAMP
6.9 approvals
id                    UUID PK
recommendation_id     UUID FK
decision              VARCHAR
approved_by           VARCHAR
reason                TEXT
created_at            TIMESTAMP
7. DATABASE RULES
Persistent state belongs in PostgreSQL.

Frontend cache is not the source of truth.

IDs should be stable.

Monetary fields use decimal/numeric types.

Never use floating point for persisted currency if avoidable.

JSONB is allowed for flexible action/simulation metadata.

Foreign keys must be respected.

Agent findings are immutable historical observations.

Recommendation status is mutable through defined transitions only.

8. DATA CONTRACT
8.1 Store
{
  "store_id": "S142",
  "city": "Bengaluru",
  "orders_per_day": 8420,
  "avg_order_value": 310.0
}
8.2 Order
{
  "order_id": "O001",
  "store_id": "S142",
  "customer_id": "C001",
  "order_value": 310.0,
  "product_margin": 75.0,
  "discount": 42.0,
  "delivery_cost": 38.0,
  "fulfillment_cost": 46.0
}
8.3 Inventory
{
  "store_id": "S142",
  "sku_id": "SKU001",
  "current_stock": 124,
  "daily_demand": 70,
  "forecast_demand": 52,
  "unit_cost": 80,
  "expiry_days": 2
}
8.4 Rider
{
  "rider_id": "R001",
  "store_id": "S142",
  "available": true,
  "orders_completed": 42,
  "avg_delivery_time": 18,
  "utilization": 0.72
}
9. AGENT CONTRACT
Every specialized agent must expose a stable callable interface.

Conceptually:

result = agent.infer(input_data)
The implementation can be synchronous internally, but the FastAPI service must not block unnecessarily during I/O.

Every agent returns the same top-level structure:

{
  "agent": "inventory",
  "store_id": "S142",
  "finding": {
    "type": "OVERSTOCK",
    "description": "Excess perishable inventory detected.",
    "severity": "HIGH"
  },
  "recommendation": {
    "action": "REDUCE_REPLENISHMENT",
    "parameters": {
      "reduction_percent": 18
    }
  },
  "financial_impact": {
    "per_order": 4.2,
    "daily": 35364,
    "monthly": 1060920
  },
  "confidence": 0.87
}
Required:

agent
store_id
finding
recommendation
financial_impact
confidence
10. SEVERITY CONTRACT
Allowed:

LOW
MEDIUM
HIGH
CRITICAL
Interpretation:

LOW: minor optimization

MEDIUM: meaningful leakage

HIGH: significant financial/operational issue

CRITICAL: immediate material risk

11. CONFIDENCE CONTRACT
Range:

0.0 ≤ confidence ≤ 1.0
Display as percentage in UI.

Example:

0.87 → 87%
Confidence is model/analysis confidence, not probability of guaranteed savings.

12. INVENTORY AGENT
Owner
P3.

Responsibilities
Demand forecasting

Overstock risk

Stockout risk

Perishable exposure

Replenishment recommendation

Input
Store + SKU + historical demand + current stock + lead time + expiry.

Processing
Historical Demand
       ↓
Forecast Demand
       ↓
Compare Current Stock
       ↓
Risk Classification
       ↓
Financial Exposure
       ↓
Recommendation
Allowed actions
REDUCE_REPLENISHMENT
INCREASE_REPLENISHMENT
FLAG_STOCKOUT_RISK
FLAG_WASTE_RISK
Output
Standard Agent Contract.

13. DISCOUNT AGENT
Owner
P3.

Responsibilities
Discount effectiveness

Purchase probability

Discount elasticity

Incremental demand

Discount ROI

Unnecessary discount detection

Processing
Discount History
       ↓
Customer / Segment Behavior
       ↓
Purchase Probability
       ↓
Incremental Demand
       ↓
Incremental Contribution
       ↓
Discount Cost
       ↓
ROI / Leakage
Allowed actions
REDUCE_DISCOUNT
REMOVE_DISCOUNT
RETARGET_DISCOUNT
KEEP_DISCOUNT
Example finding
High-frequency customers already have high organic
purchase probability; discount spend is not producing
proportional incremental demand.
14. FULFILLMENT AGENT
Owner
P4.

Responsibilities
Rider utilization

Single-order trip detection

Batching opportunities

Capacity imbalance

Fulfillment cost

Processing
Orders
+
Rider Availability
+
Delivery History
        ↓
Utilization Analysis
        ↓
Batching / Capacity Opportunity
        ↓
Cost Estimate
        ↓
Recommendation
Allowed actions
BATCH_DELIVERIES
REALLOCATE_RIDERS
ADJUST_CAPACITY
FLAG_LOW_UTILIZATION
Do not implement a full production VRP.

15. UNIT ECONOMICS ENGINE
Owner
P4, with interface consumed by P5 and backend.

Rule
This component is deterministic.

The LLM must never independently calculate contribution.

Function
Conceptually:

calculate_contribution(economic_input)
Formula
Contribution / Order =
Revenue
+ Product Margin
- Discount
- Delivery
- Fulfillment
- Dark Store
- Payment / Platform
- CAC
Output
{
  "revenue_per_order": 310,
  "product_margin": 75,
  "discount": 42,
  "delivery_cost": 38,
  "fulfillment_cost": 46,
  "dark_store_cost": 21,
  "payment_cost": 8,
  "cac": 19,
  "contribution_per_order": -99
}
Demo numbers are synthetic/modelled unless independently verified.

16. SIMULATION ENGINE
Owner
P4.

Purpose
Answer:

What happens financially if we apply this intervention?

Input
Baseline economics + proposed action + action parameters.

Output
{
  "scenario_id": "SIM001",
  "baseline": {
    "contribution_per_order": -96.6
  },
  "projected": {
    "contribution_per_order": -67.6
  },
  "impact": {
    "per_order": 29.0,
    "daily": 244360,
    "monthly": 7330800
  },
  "confidence": 0.84
}
Rules
Simulation must:

Preserve baseline.

Apply intervention assumptions.

Recalculate affected economics.

Return before/after values.

Never mutate production/demo state merely by simulating.

Use deterministic seed for the hero demo.

Clearly distinguish projected/modelled impact from actual measured impact.

17. CFO AGENT
Owner
P5.

Technology
LangGraph + LLM.

Role
The CFO Agent is the financial decision orchestrator.

It does NOT replace the specialized ML agents.

Inputs
Store state
Inventory findings
Discount findings
Fulfillment findings
Unit economics
Simulation outputs
Optional policy context
Responsibilities
Detect largest leakage.

Quantify financial impact.

Generate candidate interventions.

Request simulation.

Compare alternatives.

Consider confidence/risk.

Rank actions.

Produce final recommendation.

Explain decision in human-readable language.

18. LANGGRAPH STATE CONTRACT
Suggested state:

{
    "store_id": str,

    "store_state": dict,

    "inventory_findings": list,

    "discount_findings": list,

    "fulfillment_findings": list,

    "unit_economics": dict,

    "candidate_actions": list,

    "simulation_results": list,

    "selected_action": dict | None,

    "cfo_reasoning": str,

    "confidence": float,

    "risk": str,

    "status": str
}
The exact internal state can evolve, but required semantic fields must remain available.

19. LANGGRAPH NODES
Recommended nodes:

load_store_state
      ↓
run_inventory_agent
      ↓
run_discount_agent
      ↓
run_fulfillment_agent
      ↓
calculate_unit_economics
      ↓
identify_leakages
      ↓
generate_candidate_actions
      ↓
run_simulations
      ↓
compare_actions
      ↓
cfo_decision
      ↓
human_approval
      ↓
execute_intervention
      ↓
recalculate_economics
Parallel execution is allowed for independent specialized agents.

20. CFO DECISION RULES
The CFO should prioritize actions using:

Expected Financial Impact
× Confidence
× Feasibility
× Risk Adjustment
The exact formula may be implemented differently, but the decision must be explainable.

CFO must prefer:

Higher expected contribution improvement

Higher confidence

Lower operational risk

Feasible interventions

Interventions validated by simulation

21. CFO OUTPUT CONTRACT
{
  "recommendation_id": "REC001",
  "store_id": "S142",
  "problem": "High discount leakage",
  "recommended_action": {
    "type": "REDUCE_DISCOUNT",
    "parameters": {
      "from": 52,
      "to": 31
    }
  },
  "financial_impact": {
    "contribution_improvement_per_order": 21,
    "projected_daily_improvement": 176820,
    "projected_monthly_improvement": 5304600
  },
  "confidence": 0.87,
  "risk": "LOW",
  "reasoning": "High-frequency customers show high organic purchase probability and the simulated discount reduction improves contribution."
}
22. LLM SAFETY / GROUNDING RULES
The LLM must:

Only reason over supplied structured data.

Never fabricate financial metrics.

Never claim actual Zepto savings from synthetic data.

Never claim an intervention was executed unless backend confirms execution.

Never claim simulation output is guaranteed.

Clearly use terms such as "projected", "modelled", or "simulated" for synthetic results.

Return structured output where possible.

Fall back gracefully if the LLM is unavailable.

If LLM fails, the deterministic pipeline must still be able to return a basic recommendation or safe error.

23. OPTIONAL RAG
RAG is NOT part of the critical path.

If implemented, it should ground the CFO in:

Business policies

Discount policies

Inventory constraints

Operational rules

Financial governance rules

RAG does NOT replace:

Forecasting

Elasticity modelling

Simulation

Unit economics

Preferred approach if time permits:

PostgreSQL + pgvector
Do not introduce a separate vector database unless necessary.

24. FRONTEND CONTRACT
Required screens
Dashboard
Shows:

Selected store

Orders/day

Revenue/order

Contribution/order

Financial leakage

Agent status/findings

Analysis
Shows:

Inventory finding

Discount finding

Fulfillment finding

Severity

Confidence

₹ impact

Recommendation
Shows:

Problem

Recommended action

Reasoning

Confidence

Risk

Expected ₹ improvement

Simulation
Shows:

BEFORE → INTERVENTION → AFTER
with contribution/order and major affected metrics.

Approval
Buttons:

APPROVE
REJECT
Execution Result
Shows:

Action status

Before

After

Improvement

Timestamp

25. FRONTEND STATE RULES
Browser memory may hold current session state.

localStorage may hold:

selected store

last demo scenario

non-sensitive UI preferences

cached read-only demo results

Cookies may hold session information if authentication is implemented.

Do NOT store:

LLM API keys

database passwords

privileged secrets

Browser cache is never the source of truth.

26. API CLIENT CONTRACT
P1 should create a single API layer:

frontend/src/services/api.ts
Functions:

getStores()
getStore(storeId)
analyzeStore(storeId)
simulateRecommendation(storeId, recommendationId)
getRecommendation(recommendationId)
approveRecommendation(recommendationId)
rejectRecommendation(recommendationId, reason)
executeRecommendation(recommendationId)
Components should not scatter raw API URLs throughout the UI.

27. BACKEND SERVICE BOUNDARIES
P2 should separate:

API route
   ↓
service
   ↓
agent / economics / simulation
   ↓
database
Routes should not contain giant business-logic functions.

Suggested:

backend/app/api/
backend/app/services/
backend/app/db/
backend/app/schemas/
28. HTTPX CONTRACT
Use:

async with httpx.AsyncClient(...) as client:
    response = await client.post(...)
Use HTTPX for:

LLM API

External service calls

Any separately deployed internal service if introduced

Use timeouts.

Handle:

timeout

connection failure

non-2xx response

malformed response

Do not make unlimited retries.

29. ASYNC CONTRACT
FastAPI:

async def endpoint(...)
Use async for:

Database I/O

HTTP requests

orchestration calls that involve I/O

CPU-heavy ML work may remain synchronous or use an appropriate execution strategy.

Do not make the entire application artificially async just for aesthetics.

30. SYNTHETIC DATA CONTRACT
Owner
P5.

Rule
Demo data must be deterministic.

Use a fixed random seed:

SEED = 42
Dataset categories
stores
orders
inventory
discounts
riders
customers/customer segments
Data must intentionally contain detectable scenarios.
Examples:

Store A:
healthy economics

Store B:
discount leakage

Store C:
inventory overstock

Store D:
rider inefficiency

Store S142:
hero demo scenario
Do not generate purely random data that produces meaningless agent results.

31. HERO DEMO SCENARIO
Store
Store ID: S142
City: Bengaluru
Orders/day: 8,420
Average order value: ₹310
Baseline contribution/order: -₹96.60
These are synthetic/modelled demo values.

Leakage 1 — Discount
Current discount: ₹52/order
Target segment: high-frequency customers
Organic purchase probability: high
Modelled improvement:

~₹21/order
Recommended action:

REDUCE_DISCOUNT
₹52 → ₹31
Leakage 2 — Fulfillment
18% of orders are single-order rider trips.
Action:

BATCH_DELIVERIES
Modelled improvement:

~₹8/order
Leakage 3 — Inventory
Perishable inventory is above expected demand.
Demand is declining.
Expiry is near.
Action:

REDUCE_REPLENISHMENT
Modelled exposure/improvement:

~₹4.20/order equivalent
32. HERO DEMO DECISION
CFO receives all three.

Example ranking:

1. Reduce unnecessary discounts
   +₹21/order
   confidence 87%
   risk LOW

2. Batch deliveries
   +₹8/order
   confidence 84%
   risk MEDIUM

3. Reduce replenishment
   +₹4.20/order
   confidence 81%
   risk LOW
For the hero scenario, the recommended combined intervention may model:

₹21 + ₹8 = ₹29/order improvement
Baseline:

-₹96.60/order
Projected:

-₹67.60/order
All figures are synthetic/modelled demo values.

33. DEMO STATE MACHINE
IDLE
 ↓
ANALYZING
 ↓
AGENTS_COMPLETE
 ↓
CFO_ANALYZING
 ↓
RECOMMENDATION_READY
 ↓
SIMULATING
 ↓
SIMULATION_READY
 ↓
PENDING_APPROVAL
 ↓
APPROVED
 ↓
EXECUTING
 ↓
EXECUTED
Alternative:

PENDING_APPROVAL
 ↓
REJECTED
34. RECOMMENDATION STATUS TRANSITIONS
Allowed:

PENDING
   ↓
SIMULATED
   ↓
APPROVED
   ↓
EXECUTED
Or:

PENDING
   ↓
REJECTED
Do not allow:

EXECUTED → PENDING
REJECTED → EXECUTED
unless an explicit reset mechanism is implemented.

35. INTEGRATION CONTRACT
Rule 1
No person depends on another person's internal implementation.

Rule 2
Everyone depends only on public contracts.

Rule 3
Mocks are allowed until integration.

Rule 4
Replace mock implementation without changing public schemas.

36. MOCKING STRATEGY
P1 may use mocked API responses.

P5 may use mocked agent outputs.

P4 may use mocked agent findings.

P2 may return fixture data before real models are integrated.

Example:

MOCK
 ↓
same JSON schema
 ↓
REAL
The schema does not change.

37. INTEGRATION CHECKPOINTS
Checkpoint 1 — Hour 1
Must have:

GitHub
branches
CONTRACTS.md
DB schema
API schema
demo scenario
Checkpoint 2 — Hour 6
Must have:

Frontend
 ↓
FastAPI
 ↓
Mock analysis
 ↓
Frontend
Checkpoint 3 — Hour 10–12
Must have:

Frontend
 ↓
FastAPI
 ↓
Real ML agents
 ↓
LangGraph CFO
 ↓
Recommendation
 ↓
Frontend
Checkpoint 4 — Hour 14–16
Must have:

Recommendation
 ↓
Simulation
 ↓
Approval
 ↓
Mock execution
 ↓
Updated economics
Checkpoint 5 — Hour 20
Feature freeze.

Only:

bug fixes

UI polish

demo reliability

deployment

documentation

38. TEAM RESPONSIBILITIES
P1 — FRONTEND / PRODUCT
Own:

frontend/
Deliver:

dashboard

analysis page

recommendation page

simulation chart

approval UI

execution result

REST API client

loading/error states

Must not modify:

ML models

LangGraph

DB business logic

P2 — BACKEND / PLATFORM
Own:

backend/
Deliver:

FastAPI app

Pydantic schemas

REST endpoints

SQLAlchemy models

DB integration

service layer

HTTPX integration

Render deployment

Must not own:

model training

UI

CFO prompts

P3 — INVENTORY + DISCOUNT ML
Own:

agents/inventory/
agents/discount/
Deliver:

training scripts

inference

demand forecast

inventory risk

discount effectiveness

elasticity

discount ROI

standardized agent output

Must not modify:

frontend

FastAPI routes

LangGraph public state

P4 — FULFILLMENT + SIMULATION
Own:

agents/fulfillment/
simulation/
economics/
Deliver:

fulfillment agent

rider utilization

batching heuristic

unit economics engine

simulation engine

before/after comparison

Must not modify:

frontend

CFO prompts

API contract

P5 — CFO / TECH LEAD
Own:

cfo/
data/
contracts/
integration
Deliver:

LangGraph

CFO state

CFO nodes

decision synthesis

synthetic dataset

integration

demo scenario

deployment coordination

demo script

P5 is the integration owner.

39. NO-CONFLICT RULES
If you need to modify another person's area:

Tell the owner.

Agree on the change.

Keep the change minimal.

Document contract impact.

Do not silently rewrite their work.

Shared files requiring team agreement:

CONTRACTS.md
README.md
DB schema
API schemas
shared TypeScript/Pydantic contracts
40. DEPLOYMENT CONTRACT
Frontend
Deploy:

Vercel
Environment:

VITE_API_BASE_URL
Backend
Deploy:

Render
Environment variables:

DATABASE_URL
LLM_API_KEY
LLM_MODEL
CORS_ORIGINS
ENVIRONMENT
Never commit these values.

Database
Hosted PostgreSQL.

41. CORS
Backend must allow only the deployed frontend origin(s).

Development may allow:

http://localhost:5173
Production should use the actual Vercel origin.

42. ENVIRONMENT CONTRACT
.env.example must contain variable names only:

DATABASE_URL=
LLM_API_KEY=
LLM_MODEL=
CORS_ORIGINS=
ENVIRONMENT=development
43. OBSERVABILITY
Minimum:

FastAPI request logging

Agent execution logging

CFO execution logging

Simulation errors

LLM errors

Every major pipeline run should have an:

analysis_id
so the demo can be traced.

44. FAILURE HANDLING
If Inventory Agent fails:

mark inventory finding unavailable
continue other agents
If Discount Agent fails:

continue with remaining agents
If Fulfillment Agent fails:

continue with remaining agents
If LLM fails:

return deterministic fallback recommendation if available
or safe error
If Simulation fails:

do not approve the recommendation as simulated
If database fails:

return controlled error
The frontend must never display fake "success" for a failed operation.

45. SECURITY CONTRACT
Never commit:

API keys
DATABASE_URL credentials
tokens
passwords
private credentials
Never expose LLM API keys to frontend.

All LLM calls originate from backend.

46. FINANCIAL DATA DISCLAIMER
Unless externally verified:

All financial values shown in the prototype are
synthetic/modelled values created for demonstration.
They must not be presented as actual Zepto financial
figures or guaranteed savings.
47. DEMO SCRIPT
Step 1
Open dashboard.

Step 2
Select:

S142
Step 3
Show:

Contribution/order: -₹96.60
Step 4
Click:

Analyze Store
Step 5
Show findings:

Discount leakage
Rider inefficiency
Inventory risk
Step 6
CFO ranks interventions.

Step 7
Click:

Simulate
Step 8
Show:

BEFORE
-₹96.60/order

AFTER
-₹67.60/order

IMPROVEMENT
+₹29/order
Step 9
Click:

Approve
Step 10
Mock execution.

Step 11
Show updated economics.

48. DEMO RULE
The demo must always use the deterministic hero scenario.

Do not depend on live random generation.

Do not depend on internet availability except required LLM API calls.

Prefer cached/fallback CFO reasoning if possible.

49. 24-HOUR BUILD PLAN
Hour 0–1
Architecture lock:

GitHub

branches

contracts

DB schema

API schema

demo scenario

Hour 1–5
Parallel build:

P1 → frontend

P2 → backend

P3 → ML

P4 → simulation

P5 → data + LangGraph

Hour 5–8
First integration.

Hour 8–12
Real agents + CFO.

Hour 12–16
Simulation + approval + execution.

Hour 16–20
UI polish + deployment.

Hour 20–22
Testing.

Hour 22–24
Feature freeze + demo + pitch + backup.

50. DEFINITION OF DONE
The MVP is considered complete only when:

[ ] Frontend deployed
[ ] Backend deployed
[ ] Database connected
[ ] Store S142 available
[ ] Analysis endpoint works
[ ] Inventory agent works
[ ] Discount agent works
[ ] Fulfillment agent works
[ ] Unit economics works
[ ] LangGraph CFO works
[ ] Simulation works
[ ] Recommendation persists
[ ] Approval works
[ ] Mock execution works
[ ] Contribution recalculates
[ ] Demo numbers are deterministic
[ ] No secrets committed
[ ] Full demo works repeatedly
51. FINAL ARCHITECTURE
                         USER
                           │
                           ▼
                 ┌─────────────────┐
                 │ React Frontend  │
                 │ TS + Vite       │
                 └────────┬────────┘
                          │
                       REST API
                          │
                          ▼
                 ┌─────────────────┐
                 │ FastAPI /Render │
                 │ Async + HTTPX   │
                 └────────┬────────┘
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
      Inventory        Discount       Fulfillment
        Agent            Agent           Agent
          │               │                │
          └───────────────┼────────────────┘
                          ▼
                 ┌─────────────────┐
                 │ Unit Economics  │
                 │ Engine          │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │ LangGraph CFO   │
                 │ LLM Reasoning   │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │ Simulation      │
                 │ Engine          │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │ Recommendation  │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │ Human Approval  │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │ Mock Execution  │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │ Recalculate ₹   │
                 └─────────────────┘

                 PostgreSQL
                 = persistent state

                 Browser cache
                 = temporary UI state
52. GOLDEN RULE
PREDICTION
    ↓
ML

CALCULATION
    ↓
DETERMINISTIC PYTHON

ORCHESTRATION
    ↓
LANGGRAPH

REASONING
    ↓
LLM / CFO

COUNTERFACTUAL
    ↓
SIMULATION

GOVERNANCE
    ↓
HUMAN

PERSISTENCE
    ↓
POSTGRES

TRANSPORT
    ↓
REST
If everyone follows this contract, implementation details can change independently without breaking the system.