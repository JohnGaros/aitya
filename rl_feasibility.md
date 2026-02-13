# Reinforcement Learning Feasibility Analysis

Can the current CDS scope support state-action value estimation in the spirit of Reinforcement Learning? This document evaluates whether the seven-view scope defined in [`pragmatic.md`](pragmatic.md) allows AITYA to evaluate *actions* -- not just predict outcomes for customers or accounts.

---

## What RL Requires

A reinforcement learning framework needs four elements: **states** (observable conditions), **actions** (decisions taken), **rewards** (outcomes), and **variation in action choices** across similar states (to learn which action is better). The question is whether the current CDS views provide all four.

---

## States: Adequate

The current scope reconstructs a reasonable state representation at any point in time:

| State Signal | Source | Quality |
|---|---|---|
| Days past due | `I_OperationalAcctgDocItem` (`NetDueDate` vs. current date) | Clean, deterministic |
| Outstanding amount | `I_OperationalAcctgDocItem` + `I_GLAcctBalance` | Reconciliation-backed |
| Current dunning level | `I_DunningHistory` | Clean |
| Historical payment behavior | `I_OperationalAcctgDocItem` (past `ClearingDate` - `NetDueDate` deltas) | Clean |
| Customer segment context | `I_Customer` (geography, industry, payment terms via `T052`) | Usable |
| Dunning velocity | `I_DunningHistory` (time between escalation events) | Clean |

A reasonable state vector s(t) for a customer-invoice pair can be encoded at any dunning decision point.

---

## Rewards: Clean

`ClearingDate` from `I_OperationalAcctgDocItem` gives an unambiguous outcome signal. Whether and when cash arrived is observable. Combined with amounts, reward functions include:

- Cash recovered within N days of action
- Discount capture rate (did the customer take the early-payment discount?)
- Terminal state: cleared vs. written off

This is better than many RL domains -- the reward signal is not noisy or delayed in the usual sense. It is observable and tied to a financial fact.

---

## Actions: This Is Where It Falls Apart

**The only observable action in the current CDS scope is dunning level escalation.** `I_DunningHistory` records when the system moved a customer to dunning level 1, 2, 3, or 4. That is the entire action space.

Three problems:

### 1. Dunning is not a decision -- it is an automation

In most SAP implementations, dunning runs are batch processes that execute the `T047`-configured rules mechanically. Customer at level 1 for more than X days and amount exceeds Y threshold → escalate to level 2. The "action" is a deterministic function of the state and the configuration parameters. An RL framework that treats a deterministic policy as its action space is evaluating a fixed rule, not learning an optimal strategy.

### 2. The real actions are invisible

The collection decisions with genuine decision value are not captured in any of the seven views:

| Action | Where It Lives | In Scope? |
|---|---|---|
| Send dunning notice | `I_DunningHistory` | **Yes** |
| Call the customer | CRM activity logs | No |
| Negotiate a payment plan | FSCM Collections Management | No |
| Offer a discount extension | Manual / no system record | No |
| Escalate to legal | FSCM Collections Management | No |
| Adjust credit limit | Customer master (manual change) | No |
| Write off the balance | `I_JournalEntryItem` (observable as a posting, not as a decision) | Partially |

The dunning notice is the *least interesting* action in the collections workflow. It is the one action that requires no judgment.

### 3. The action space is degenerate

Dunning levels are ordinal and monotonic -- you can only escalate, never de-escalate. The policy is "wait → escalate → wait → escalate → write off." There are no branching decisions, no action alternatives at any given state. In RL terms, the policy is a single trajectory through a one-dimensional action space. Q(s, a) collapses to Q(s, "escalate") vs. Q(s, "wait"), which is a timing prediction -- when will escalation produce payment? That is a supervised regression problem, not RL.

---

## Variation: The Federated Angle

The one place where genuine policy variation exists is **cross-tenant**. Different tenants configure different dunning procedures via `T047`:

- Tenant A: 4 levels, 14-day intervals, €100 minimum
- Tenant B: 3 levels, 30-day intervals, €500 minimum

If outcomes are observed under both policies for customers in similar states (same industry, similar aging, similar amounts), something closer to **offline policy evaluation** becomes possible: which dunning configuration produces better cash conversion for a given customer profile?

But this is confounded by everything that cannot be observed:

- The tenants serve different customer bases
- Different economic conditions per geography
- Different unobserved collection actions (phone calls, negotiations)
- Different credit policies upstream (who they extend credit to in the first place)

The comparison is not of actions in controlled states -- it is of entire business environments, with the difference attributed to the dunning config. That is a causal inference problem, not an RL problem, and it is deeply confounded. Cross-tenant dunning policy evaluation is analyzed in detail in [`pragmatic.md`](pragmatic.md) under Federated Benchmarking, including its risks and limitations.

---

## What Could Honestly Be Done

Dropping the RL framing and asking "what action-related questions can the data actually answer?" yields two honest answers:

### 1. Dunning policy evaluation (cross-tenant, offline)

Given the federated network, estimate: "For customer-state profiles like X, does a 14-day dunning interval produce faster clearing than a 30-day interval?" This is **contextual bandit-style estimation** at best -- one action dimension (dunning timing/aggression), observed under natural variation across tenants. Useful, but a statistical comparison of configured policies, not learned optimal actions. See [`pragmatic.md`](pragmatic.md) for the full treatment.

### 2. Dunning timing prediction (single-tenant, supervised)

Given a customer's state (aging, amount, history, dunning level), predict the probability of payment within N days. This informs *when* human intervention is worth the effort -- a prioritization signal. It is the collection scoring already described in [`pragmatic.md`](pragmatic.md). Calling it "RL" adds no value over calling it what it is: supervised prediction.

---

## What Would Be Needed for Real State-Action Value Estimation

To actually estimate Q(s, a) -- the value of taking action a in state s -- the following would be required:

### 1. Observable action diversity

Multiple distinct actions taken in similar states: called, emailed, offered payment plan, escalated to legal, did nothing. This requires FSCM Collections Management logs or an external CRM system -- neither is in the current scope.

### 2. Action timestamps

Not just "what dunning level" but "what specific intervention happened on what date." The current views give dunning events but not the richer action vocabulary.

### 3. Counterfactual variation

Cases where similar customers in similar states received different treatments. This requires either randomized experimentation (unlikely in collections) or sufficient natural variation in human decisions (which exists in manual collections but is not recorded in the current scope).

### 4. Off-policy estimation methods

Since the learning would be from logged actions (not a live policy), importance sampling, doubly robust estimators, or conservative Q-learning would be necessary. These are methodologically mature but data-hungry, and require the action diversity and counterfactual variation described above.

---

## Verdict

The current CDS views support **predicting payment outcomes given states** (supervised learning) and **comparing dunning configurations across tenants** (offline policy comparison). They do not support genuine state-action value estimation because the observable action space is a single deterministic automation with no branching, no alternatives, and no human decision signal.

| Capability | Feasible? | Method | Limitation |
|---|---|---|---|
| Predict which customers will pay late | **Yes** | Supervised learning on state features | No action optimization |
| Prioritize collection effort | **Yes** | Risk scoring from payment behavior prediction | Prioritization, not action recommendation |
| Compare dunning configurations across tenants | **Partially** | Offline policy evaluation / contextual bandit | Confounded by unobservable differences |
| Recommend specific collection actions | **No** | Would require RL with diverse action space | Actions not recorded in current scope |
| Optimize full collection workflow | **No** | Would require online/offline RL with FSCM data | Outside current architecture |

The honest product claim is: "We predict which customers will pay late and prioritize your collection effort." The cross-tenant dunning policy evaluation adds a prescriptive layer for dunning configuration specifically. Neither claim requires RL, and framing them as RL would be technically misleading.
