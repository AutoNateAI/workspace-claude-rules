# Architectural PR Review with FigJam Diagrams

Perform a deep architectural review of a PR by analyzing the contract (before and after), mapping the blast radius across all affected components, and producing FigJam-ready diagrams with detailed narrative explanations.

## When to Use

Attach this rule when you say things like:
- "Review this PR architecturally"
- "What's the blast radius of this PR?"
- "Give me a contract analysis of this change"
- "Do a deep review with diagrams"
- "FigJam review of PR #XXXX"

---

## Philosophy: Contract-First Review

Every meaningful PR changes a **contract** — the agreement between components about what data flows where, in what shape, and under what conditions. Before looking at code quality or style, answer:

1. **What was the contract before?** What data flowed, who produced it, who consumed it, what were the invariants?
2. **How did the contract change?** What paths were added, severed, or simplified? What assumptions broke?
3. **What is the blast radius?** Every consumer of the old contract is potentially affected. Map them all.

**The review question is never "does this code work?" — it's "does every consumer of this contract still get what it needs?"**

---

## Process

### Step 1: Gather the PR

```bash
# Get PR metadata
gh pr view {PR_NUMBER} --repo {OWNER}/{REPO}

# Get full diff
gh pr diff {PR_NUMBER} --repo {OWNER}/{REPO}

# Get list of changed files
gh pr view {PR_NUMBER} --repo {OWNER}/{REPO} --json files --jq '.files[].path'
```

### Step 2: Identify the Contract

Read every changed file. For each one, identify:

- **What data does this component produce?** (exports, props passed down, store mutations, API calls)
- **What data does this component consume?** (imports, props received, store reads, API responses)
- **What are the invariants?** (conditional rendering, type guards, capability checks, feature flags)

Build a mental model of the **dependency graph** — not just file imports, but data flow. A component that reads from a MobX store depends on that store even without a direct import.

### Step 3: Map the Blast Radius

For every contract change, trace **all consumers**:

1. **Direct consumers** — Components that import/use the changed code
2. **Indirect consumers** — Components that depend on data shaped by the changed code
3. **Behavioral consumers** — UI flows or user experiences that relied on the old contract
4. **Persistent consumers** — Backend data, saved configs, or cached values tied to the old contract

Classify each affected item:

| Status | Meaning | Color |
|--------|---------|-------|
| **Deleted** | Code or file completely removed | Red `#ff6b6b` |
| **Modified** | Code changed but still exists | Yellow `#ffd93d` |
| **Unchanged** | Code untouched but behavior affected | Green `#6bcb77` |
| **Created** | New code introduced | Blue `#74b9ff` |

### Step 4: Classify Edge Changes

Edges (data flow paths) matter more than nodes. For every edge in the dependency graph:

| Edge Status | Meaning | Style |
|-------------|---------|-------|
| **Severed** | Data no longer flows on this path | Red dashed line |
| **Simplified** | Path still exists but carries different/reduced data | Yellow solid line |
| **Unchanged** | Path works exactly as before | Green solid line |
| **New** | Path that didn't exist before | Blue solid line |

### Step 5: Generate Diagrams

Produce **3 diagrams** as mermaid code blocks. These are written in mermaid syntax but rendered in FigJam (user pastes via `/mermaid` in FigJam).

#### Diagram 1: Component Call Stack & Props Flow

Show the full component hierarchy with:
- Each component as a node (colored by status)
- Props passed between components as labeled edges
- Store reads/writes as edges to the store node
- Edges colored/styled by their change status (severed, simplified, unchanged)
- Function signatures and param types on relevant nodes
- Use `linkStyle` directives for edge coloring

```
flowchart TD
    classDef deleted fill:#ff6b6b,stroke:#cc0000,color:#000
    classDef modified fill:#ffd93d,stroke:#cc9900,color:#000
    classDef unchanged fill:#6bcb77,stroke:#28a745,color:#000
    classDef created fill:#74b9ff,stroke:#0984e3,color:#000

    COMPONENT["📱 ComponentName — file.tsx\nKey details about what changed"]:::modified

    %% Use linkStyle for edge coloring
    linkStyle 0 stroke:#cc0000,stroke-width:3px,stroke-dasharray:5
```

#### Diagram 2: Utility / Dependency Graph

Show the support layer (utilities, helpers, shared types, enums) and every call site:
- Each utility function with signature as a node
- Each call site as a node
- Edges from function to call site, colored by status
- Highlight cases where **both ends** of an edge were deleted vs. only one end

#### Diagram 3: Before vs After Data Flow

Side-by-side subgraphs showing:
- **Before:** The full data cycle (write paths → store → read paths → UI → user action → write paths)
- **After:** What survived, with severed paths shown as dashed
- **Risk callout box:** Key review questions

Include `linkStyle` directives to make the **one surviving path** thick green, severed paths red dashed, and simplified paths yellow.

### Step 6: Write Narrative Explanations

**Every diagram MUST have a detailed narrative.** The diagram alone is not enough. For each board, write:

#### Contract Before
- Describe in plain English what data flowed where and why
- Name the write paths (how data entered the system)
- Name the read paths (how data was consumed)
- Explain the invariants (what conditions gated behavior)

#### What Changed
- For each write path: was it severed, simplified, or unchanged?
- For each read path: was it severed, simplified, or unchanged?
- Call out **asymmetric treatment** (e.g., one feature ungated but another deleted)

#### Key Severed Edges
- Identify the 2-3 most consequential broken connections
- Explain why they matter (data loss, behavioral change, stale data)
- Use the format:
```
Source → intermediate → destination
```
And explain what used to flow on that path.

#### Review Questions
- Raise specific questions about ambiguous changes
- Flag potential product decision vs. accidental side effect
- Note backend/persistence implications

### Step 7: Create Output Files

Save all artifacts to the daily notes structure:

```
daily-notes/YYYY/MM/week-NN/DD/pr-reviews/reviewing/
├── pr-XXXX-description.md            # Full analysis doc
├── pr-XXXX-board-narratives.md        # Diagram explanations
```

The analysis doc should contain:
1. PR metadata (author, repo, additions/deletions, labels)
2. The Contract Before (with table of all touchpoints)
3. What Changed (with before/after comparison table)
4. All 3 mermaid diagram code blocks (for easy copy-paste to FigJam)
5. Risk Assessment (low/medium/high for each concern)
6. Review Questions (numbered, specific, actionable)
7. Verdict (one paragraph summary of review stance)

The board narratives doc should contain:
1. How to read the diagrams (color legend, edge legend)
2. For each board: Contract Before → What Changed → Key Severed Edges
3. Expanded review questions with full reasoning

### Step 8: Update Session Log

Add an entry to the session log noting:
- Which PR was reviewed
- Contract summary (before → after in one line each)
- Key files and count
- Top review question
- Link to the analysis doc

---

## Mermaid Rendering in FigJam

**Important:** These diagrams are authored in mermaid syntax but rendered in Figma's FigJam whiteboard tool.

**How the user pastes them:**
1. Open a FigJam board
2. Type `/` to open quick actions
3. Select **"Mermaid"**
4. Paste the mermaid code block (without the triple backtick fences)
5. FigJam renders it as a visual diagram on the board

**Mermaid tips for FigJam rendering:**
- Keep node labels concise but include function signatures and key details
- Use `\n` for line breaks within node labels
- Use `classDef` for node coloring (deleted=red, modified=yellow, unchanged=green, created=blue)
- Use `linkStyle N stroke:#color,stroke-width:Npx` for edge coloring
- Use `stroke-dasharray:5` for dashed lines (severed edges)
- Subgraphs render as labeled boxes — use for Before/After comparisons
- Test that link indices match edge order (edges are numbered 0-based in declaration order)
- Avoid special characters in labels that break mermaid parsing (`<`, `>`, `{`, `}` in text need escaping or use unicode alternatives like `‹›`)

**Diagram sizing guidance:**
- Board 1 (call stack): Can be large — this is the main architectural view
- Board 2 (dependencies): Medium size — focused on one layer
- Board 3 (before/after): Side-by-side subgraphs — keep each side balanced

---

## Example Review Questions by Category

### Contract Questions
- Was [feature] removal intentional or collateral from removing [other feature]?
- Does the backend handle the new default value correctly?
- Are there stale configs/data that reference the old contract?

### Behavioral Questions
- Can users still access [feature] after this change?
- What happens to in-flight requests during deployment?
- Does the UI correctly reflect the new state?

### Persistence Questions
- Are there saved records with old values that won't be read/updated/cleared?
- Will a migration be needed if this contract is ever re-added?
- Do analytics events still fire correctly with the new data shape?

---

## Integration

- Uses daily-notes-structure.md for file placement
- Complements pr-review-analysis.md (that rule handles comment triage; this rule handles architectural review)
- Diagrams follow mermaid-diagrams.md conventions
- Session log updates follow session-logging.md
- PR feedback follows pr-feedback-workflow.md for posting comments
