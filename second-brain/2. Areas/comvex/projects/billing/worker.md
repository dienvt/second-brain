## Publisher


## Job Handler
First we have to `registerWorkerRoutes`

Check the current Segment model
 -

Queue batch segment calculator
```mermaid
flowchart TD
    A([Start]) --> B[CLI: UpsertBillingSegmentsDispatcherCommand]
    B --> C{Parse args<br/>accounts, from, to, features}
    C --> D[Query eligible accounts include non-demo, enabled/changed]
    D --> E{Account has feature flag<br/>BILLING_SEGMENT_CALCULATION_WITH_QUEUE?}
    E -- No --> F[Skip account] --> H
    E -- Yes --> G[Dispatch MakeSegmentJob args accountId, from, to, feature per feature] --> H[Queue]
    H --> I[Worker dequeues MakeSegmentJob]
    I --> J{Find Account by ID}
    J -- Not found --> K([End])
    J -- Found --> L[ConnectToAccountDatabase accountId]
    L --> M[Construct SegmentMaker args account, feature, from, to]
    M --> N[Initialize: load billingProperties, validate tiers/increment, cache cost/pro-rata flag]
    N --> O{Feature == pricing_plan<br/>AND no account.pricing_plan?}
    O -- Yes --> P[Skip segment creation] --> Q([End])
    O -- No --> R[CalculateCost by feature]
    R --> S{Free months?}
    S -- Yes --> T[Zero out base & additional cost]
    S -- No --> U{Subscription-based AND no active subscriptions?}
    U -- Yes --> T
    U -- No --> V[Keep computed costs]
    T --> W[Compute pro-rata % first month only]
    V --> W
    W --> X[UpsertSegmentJob: total = ceil base+additional *proRata/100 ]
    X --> Y[DiscountCalculator: validate settings & window; compute fixed/percentage]
    Y --> Z[Create/Update Segment ]
    Z --> AA([End])
```




1. Pricing plan model is just a template, the account pricing plan will be the associated billing plan for account. And we copy the properties of the template to it with a freedom to modify for a specific accnt

# Prorated?
