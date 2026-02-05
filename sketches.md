# Sketches

Ideas before the score. Fragments to develop.

---

## Instrumenting for Questions You Don't Know to Ask

Workers return structured results—success, failure, findings. The structure encodes assumptions about what matters. A researcher returns "recommendations," a reviewer returns "approved/rejected." These categories constrain discovery.

The interesting signal often lives outside the schema. A worker might notice something orthogonal to its task—a pattern, a smell, a hunch. Current design: that gets dropped. No channel for "by the way, something feels off about this whole approach."

### Three patterns worth exploring:

**The Sidebar** — Optional `## Observations` section in worker output. Low-friction invitation to note what doesn't fit the main schema. Workers decide what's notable.

**The Scratchpad** — Stream reasoning, not just conclusions. Manager (or sensemaking agent) scans for patterns across workers. Expensive but captures pre-filtered observations naturally.

**The Devil's Advocate** — Before completing, workers answer: "What might I be missing? What could go wrong with my recommendation? What did I notice that wasn't my job to investigate?"

### Open questions:

- Signal vs. noise: how do you keep observations useful without drowning in irrelevant hunches?
- Workers are told "stay in scope"—does an observations channel undermine focus, or create healthy tension?
- Who interprets? Manager? Dedicated sensemaking agent? Periodic human sweep?
- If an observation proves valuable, how does that feedback reinforce future observation quality?

---

## Decision Half-Life

Which parts of this design will rot first? The Manager/Worker boundary is load-bearing. If workers start making meta-decisions, or manager starts implementing, the pattern collapses.

Worker prompt templates encode assumptions about scope that will age. What seams would make replacement painless?

---

## Forgetting as Architecture

Workers are stateless by design—report back and vanish. But the Manager accumulates context. What happens when that context becomes stale or contradictory?

The INCAPABLE response is a form of forgetting—worker says "this isn't for me" and the system routes around it. What's the equivalent at the Manager level? When does a Manager need to forget its own plan?

---

- What if workers could leave notes for future workers on the same codebase?
- Anomaly detection across worker outputs: "three workers mentioned the auth module smells weird"
- The value of Event Storming isn't the artifact—it's the conversation. What if workers talked to each other?
