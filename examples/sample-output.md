# Sample Output

**Scope:** full repository read, default exclusions applied (`node_modules/`, `dist/`, `build/`, `.next/`, lock files, binary fixtures).

**Subagents:** 7 / 7 returned (default mode). Complete coverage achieved; no files skipped.

### Executive Summary

🟡 Functional but fragile overall. **Reliability and error handling** is the weakest dimension at 🟠 2 — two separate service modules share the same bug-hiding `except` pattern. Architecture and maintainability both sit at 🟡 3, dragged down by one oversized frontend component. Extensibility, security, and testing are healthy (🟢 4). No Critical findings in this pass. The default 7-subagent inspection ran without degradation and every in-scope file was read in full.

### Dimension Scorecard

- **Architecture and module boundaries** — 🟡 3 Functional but fragile
  Rationale: `MapView.tsx` tangles four unrelated responsibilities (H-1); module seams elsewhere in `frontend/src/` are clean.
  Scored by: Subagent-architecture, main-thread-verified.

- **Maintainability** — 🟡 3 Functional but fragile
  Rationale: One oversized hot-path file (H-1) dominates. Surrounding code is readable, consistently styled, and passes lint.
  Scored by: Subagent-maintainability, main-thread-verified.

- **Extensibility** — 🟢 4 Healthy
  Rationale: Provider aggregation and layer registry are pluggable; direct inspection of `backend/app/services/providers/*.py` and `frontend/src/map/layers/*.ts` confirms new sources/layers can be added without core edits. No open findings in this dimension.
  Scored by: Subagent-extensibility, main-thread-verified.

- **Reliability and error handling** — 🟠 2 Significant debt
  Rationale: Broad `except Exception` in `overview.py` hides programmer errors behind "resilience" (H-2); the same pattern survives in `event_geocoder.py` (M-2). Two separate files repeating the same anti-pattern is the clearest form of structural debt in this category.
  Scored by: Subagent-reliability proposed 🟡 3; main-thread override to 🟠 2. Reason: the subagent scored each file independently; the override reflects that the pattern repeats across files, which is worse than one instance.

- **Security** — 🟢 4 Healthy
  Rationale: No hardcoded credentials, auth middleware consistent across routers, input validation at request boundaries. Direct inspection of `backend/app/auth/*.py` and `backend/app/routers/*.py` found no exploitable paths. No open findings in this dimension.
  Scored by: Subagent-security, main-thread-verified.

- **Testing and engineering quality** — 🟢 4 Healthy
  Rationale: Regression tests cover hot paths; CI gates correctness (type-check, unit, integration), not only style. Only soft spot is doc-dependent local-dev setup (M-1).
  Scored by: Subagent-testing, main-thread-verified.

- **AI-specific project risks** — 🟡 3 Functional but fragile
  Rationale: `prompts/` and `skills/` are organized and versioned in git, but no evals or goldens guard prompt changes — a bad prompt edit can ship silently. Prompt-version ownership is implicit.
  Scored by: Subagent-ai-risks, main-thread-verified.

### Top Findings

#### Critical

✅ No Critical issue was found in this pass.

#### High

🔴 **H-1 — `frontend/src/components/MapView.tsx` owns four unrelated responsibilities in one 6 200-line file.**

- Severity: High · Blocking: no · Confidence: high · Verification: main-thread-verified
- Location: `frontend/src/components/MapView.tsx:1-6200` (whole file inspected via split passes)

Evidence (`frontend/src/components/MapView.tsx:1-40`, top of file):

```tsx
export function MapView(props: MapViewProps) {
  // map lifecycle
  useEffect(() => { initMap(containerRef.current, props.config); return () => destroyMap(); }, []);
  // layer coordination
  const layers = useLayerRegistry(props.layers);
  // interaction routing
  const dispatch = useInteractionDispatch(layers, props.onEvent);
  // panel coupling
  const panelState = usePanelSync(props.panel, layers, dispatch);
  // ...
```

Evidence (`frontend/src/components/MapView.tsx:4820-4835`, mid-file — same four concerns still interleaved):

```tsx
function handleCanvasClick(e: MouseEvent) {
  persistLastClick(e);                     // persistence
  dispatch({ kind: 'select',               // interaction routing
             payload: resolveHit(e, layers) });
  requestPanelRefresh();                   // panel coupling
  scheduleLifecycleTick();                 // map lifecycle
}
```

File totals 6 200 lines (`wc -l`); the four responsibilities above interleave throughout the split passes that covered the whole file.

Why it matters: four distinct concerns share one module. Unrelated behavior changes land in the same diff and cannot be isolated for review, rollback, or regression bisection. Every future map change has a higher chance of accidental behavior drift.

Recommendation: extract layer coordination (`useLayerRegistry`) and panel sync (`usePanelSync`) into sibling components that only talk to `MapView` through props; keep the root at lifecycle and interaction routing. Do this before adding new features in this file.

🔴 **H-2 — `backend/app/services/overview.py` tangles fallback, aggregation, and output shaping; broad `except` hides real failures.**

- Severity: High · Blocking: yes · Confidence: high · Verification: cross-subagent-corroborated (architecture + reliability)
- Location: `backend/app/services/overview.py:450-930`
- CWE: [CWE-755](https://cwe.mitre.org/data/definitions/755.html) (Improper Handling of Exceptional Conditions)

Evidence (`backend/app/services/overview.py:612-640`):

```python
try:
    providers = [p.fetch(region) for p in self.providers]
    result = aggregate(providers, boundary=lookup_boundary(region))
    return shape_for_api(result)
except Exception:
    # degraded mode — keep the UI alive
    return EMPTY_OVERVIEW
```

Why it matters: a bare `except Exception` at this layer swallows programmer errors (e.g. `KeyError` from a typo) and upstream transport failures indistinguishably, and returns the same empty payload as a legitimately empty region. Observability, alerting, and user-visible "no data here" states all collapse into one branch. This is how bugs get hidden behind "resilience".

Recommendation (in preferred order): (a) narrow the except to transport/timeout errors only and re-raise the rest so tests and sentry catch them; (b) split fallback, aggregation, and output shaping into three functions so each one's failure mode is localized; (c) return a discriminated result (`Empty | Degraded | Ok`) so the UI can distinguish "nothing here" from "upstream down".

#### Medium

🟡 **M-1 — `frontend/src/lib/api.ts` collaborator setup depends on README accuracy.**

- Severity: Medium · Blocking: no · Confidence: medium · Verification: subagent-only
- Location: `frontend/src/lib/api.ts:11-15`

Evidence (`frontend/src/lib/api.ts:11-15`):

```ts
export const API_BASE =
  import.meta.env.VITE_API_BASE ??
  // see README §Local Dev for the expected fallback
  "http://localhost:8000";
```

Why it matters: the implementation is cleaner than before, but the default URL and its relationship to README instructions mean that if docs drift, new collaborators hit integration failures that *look* like backend bugs. The code silently encodes a contract that lives elsewhere.

Recommendation: add a startup log line that echoes the resolved `API_BASE` and its source (env var vs. default), so the contract becomes self-describing at runtime instead of doc-dependent.

🟡 **M-2 — `backend/app/services/event_geocoder.py` degraded-mode boundary survives only by convention.**

- Severity: Medium · Blocking: no · Confidence: medium · Verification: main-thread-verified

Evidence (`backend/app/services/event_geocoder.py:130-170`, abbreviated):

```python
def geocode(event):
    try:
        return _resolve(event)        # transient upstream failure → fall back
    except UpstreamTimeout:
        return None
    except Exception as exc:          # programmer errors leak here today
        logger.warning("geocode failed: %s", exc)
        return None
```

Why it matters: transient failures and programming errors are better separated than before, but the broad `except Exception` is one refactor away from reintroducing the same "hide the bug" pattern as H-2. Future maintainers cannot tell from the code alone that the second branch is intended to be a last-resort, not a general safety net.

Recommendation: replace the trailing `except Exception` with an explicit list of known failure classes, and let anything else crash the task so it surfaces in error reporting. Same root pattern as H-2 — fixing one without the other leaves the broader invariant fragile.

### Strengths

✅ All 7 subagents returned; complete coverage achieved without entering over-budget degraded mode.

✅ Regression tests cover hot-path behavior; CI gates correctness rather than style alone.

✅ Frontend is moving toward better module boundaries outside of `MapView.tsx`.

✅ Provider and layer abstractions make data sources genuinely pluggable (see Extensibility scorecard entry).

### Residual Risks

⚠️ Scope exclusions applied: `node_modules/`, `vendor/`, `dist/`, `build/`, `.next/`, lock files, and binary fixtures were excluded per Coverage Rules §3. The scorecard scores reflect in-scope code only.

⚠️ AI-evaluation paths (prompt goldens, version ownership) are under-tested today. If `prompts/` and `skills/` keep growing, add evals before the next audit or the AI-risks score will drift downward.

⚠️ Reliability scored 🟠 2 based on two files sharing the same anti-pattern. If additional service modules adopt the same broad-`except` style, expect the score to move toward 🔴 1.

### Verdict

🟡 The project is worth continuing, but reliability is the dimension that should not be allowed to slip further. Priority next steps:

1. Fix **H-2** first — narrowing the broad `except` in `overview.py` closes the largest active "bugs hidden behind resilience" surface and is a small diff.
2. Apply the same pattern fix to **M-2**; leaving it half-done keeps the invariant fragile and preserves the 🟠 2 reliability score.
3. Split **H-1** before adding more behavior to `MapView.tsx`. Non-blocking for release; blocking for anyone who needs to change map behavior safely.
