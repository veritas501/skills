# Plan

Skills for turning ideas into plans humans can review and agents can execute without drifting.

A plan moves through three phases — discuss, write, execute — each its own skill. Discussion stays breadth-first so the top-level frame stabilises before any detail; the plan is then laid out as a layered multi-file directory (`.plans/<MM-DD>-<feature_name>/`) where top-level design is split by dimension and each task references the design it must obey; execution treats that design as a hard gate rather than reference material.

- **[plan-discuss](./plan-discuss/SKILL.md)** — Breadth-first discussion phase: confirm the top frame first, raise one point at a time, probe a branch's feasibility before drilling into it, fold detail back to a summary when the frame changes. Hands off to `plan-write` once the frame is frozen.
- **[plan-write](./plan-write/SKILL.md)** — Lay an agreed design into a layered multi-file plan directory instead of one `plan.md`: top-level design split by dimension (`00-overview` … `50-risks`), each subtask its own file carrying a `Depends-on:` header so the design has enforcing power.
- **[plan-execute](./plan-execute/SKILL.md)** — Execute a `plan-write` directory task-by-task with the top-level design files as a hard precondition: re-read each task's `Depends-on:` files before coding, check output against them after, and STOP to escalate when implementation reality conflicts with the design instead of drifting silently.
