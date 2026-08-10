
  2. Worker logs. The breaker logs at each state transition, so in your log aggregation you can search for:
  - "sync circuit open, skipping sync" — every time a sync is skipped for a tripped connection (includes connectionId and account)
  - the recordSyncCircuitFailure warn lines — failures being counted toward the threshold

  A saved log query or alert on those strings gives you passive monitoring without touching Redis at all — that's the quickest thing you could set up today, and it also works retroactively
  (you can see when a breaker tripped, which Redis can't tell you after the TTL expires).