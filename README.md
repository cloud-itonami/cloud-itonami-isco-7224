# cloud-itonami-isco-7224

Open Occupation Blueprint for **ISCO-08 7224**: Metal Polishers, Wheel Grinders and Tool Sharpeners.

This repository designs a forkable OSS business for a metal-polishing/wheel-grinding/tool-sharpening workshop scheduling and logistics coordination practice: a workshop scheduling and supply-coordination robot manages crew/task records under a governor-gated actor, so a metal polishing/grinding/sharpening crew keeps its own operating records instead of renting a closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/metalpolisher/` implements the
`MetalPolisherActor` as a `langgraph.graph/state-graph`
(`metalpolisher.actor`) wired to a `Metal Polisher Advisor`
(`metalpolisher.advisor`) and an independent `MetalPolisherGovernor`
(`metalpolisher.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok?) +-> :request-approval (:escalate?, human-in-the-loop interrupt)
+-> :hold (:hard?)`. 21 tests / 45 assertions green (`kbb -M:test`).
HARD invariants (always hold, never overridable): worker provenance,
workshop provenance, no-actuation (`:effect` must be `:propose`), a closed
op-allowlist (`:log-work-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — nothing else may
ever be proposed), and a permanent, unconditional block on any
proposal that would directly finalize a grinding/polishing-execution
decision (e.g. deciding to proceed with a specific grinding or
polishing operation) or override a shop safety officer's judgment.
Always-escalate paths (human sign-off regardless of confidence, mapping
this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above
the registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a workshop scheduling/logistics coordination robot performs crew scheduling, task/materials-usage/progress-record logging and abrasive-materials/tooling supply-order coordination for a metal polishing/grinding/sharpening crew, under an actor that proposes actions and an independent **Metal Polisher Governor** that gates them. The governor never
dispatches hardware itself, never performs grinding or polishing work on the shop floor, and never finalizes a grinding/polishing-execution decision or overrides a shop safety officer's judgment; `:high`/`:safety-critical` actions (such as a flagged abrasive-wheel-hazard/particulate-exposure/hand-eye-injury concern, or an above-threshold supply order) require human sign-off. **This actor coordinates workshop scheduling/logistics only — it never performs grinding or polishing work itself.**

## Core Contract

```text
crew roster + workshop registration + safety-reporting policy
        |
        v
Metal Polisher Advisor -> Metal Polisher Governor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, finalize
a grinding/polishing-execution decision, override a shop safety officer's
judgment, suppress an operating record, or disclose sensitive data
without governor approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7224`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
