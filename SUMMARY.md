# Duplicate Resolution — Design Summary

## Approach

I designed a single-screen Queue Overview that lets a RevOps analyst triage 342 duplicate pairs without navigating between pages. The layout is a split-pane: a scrollable pair list on the left (40%) and a persistent detail view on the right (60%). Clicking a pair in the list instantly shows both records side by side — no modals, no page transitions, no lost context.

This mirrors the mental model of email triage: scan the inbox, review the message, act, move on. For an analyst working through hundreds of pairs in a session, every eliminated click compounds.

The design recognises that most RevOps workflows are bulk-first. Kernel's AI has already done the analysis — the analyst's job is to clear the safe pairs fast and focus time on the ambiguous ones. The risk tier filter combined with a contextual "Merge all X pairs" button lets an analyst clear 140+ Very Low risk pairs in a single click, then drill into Medium/High risk pairs individually.

## Key Decisions

**Risk tiers over raw confidence scores.** The brief provides AI confidence percentages, but a number like "78%" doesn't tell an analyst what to *do*. I mapped confidence to Kernel's existing risk tier system (Very Low / Low / Medium / High) so the score becomes an action: "Very Low = safe to bulk merge" vs "High = review carefully." The risk score (100 minus confidence) is still shown for precision. The risk dropdown includes colour-coded dots and pair counts per tier for quick orientation.

**Bulk-first workflow.** Filtering by risk tier reveals a contextual "Merge all N pairs" button inline with the toolbar. This is the primary action path — not reviewing individual pairs. The detail pane exists for the 20% of pairs that need human judgement, not the 80% that Kernel's AI has already resolved with high confidence.

**Split-pane over modal.** The standard pattern is list → click → modal → close → repeat. At 342 pairs that's 684 open/close actions. The persistent detail pane eliminates this entirely. Arrow keys (j/k) let power users work through the queue without touching the mouse.

**Stacked names with merge arrow.** I explored four treatments for showing the pair relationship (inline arrow, stacked, split pill, mini cards). Stacked won because it's the only pattern that scales to multi-duplicate clusters — if one account has 10 matches, they list vertically without breaking the layout. A merge arrow icon marks the duplicate record.

**Risk chips as the single colour signal.** Early iterations had coloured chips for both risk tiers (green/blue/yellow/red) and classification types (red/blue/purple/amber). Two colour systems competing for attention creates noise. Classification badges became neutral grey with a light border — they're categorical labels, not urgency indicators. Risk carries the colour because it drives the action.

**AI suggestion matches risk context.** The Kernel Suggestion card in the detail pane changes colour to match the risk tier — green for Very Low (confident merge), yellow for Medium (review needed), red for High. This reinforces trust: the system's recommendation is visually consistent with its confidence level.

**One screen, not a dashboard + detail page.** The brief asks for a Queue Overview. I kept everything in one view: toolbar with filters, search, and progress for the landscape; list for navigation; detail for decision-making. No separate dashboard screen. A RevOps analyst's time is spent resolving pairs, not admiring charts.

## Trade-offs

**No field-level merge picker.** The detail pane shows both records' fields side by side but doesn't let the analyst pick which value to keep per field. Kernel's AI recommendation already suggests a primary record, which covers most cases. Adding per-field selection would be a natural evolution for Medium/High risk pairs where the merge decision is less clear-cut.

**No diff highlighting.** Fields that differ between records aren't visually highlighted. Adding a subtle background to mismatched rows would help the analyst spot decision points faster. Kept it out to avoid scope creep but it's a quick win.

**Fixed risk thresholds.** The tier boundaries are hardcoded. In production, these should be configurable per-customer, matching Kernel's existing risk tier settings page.

**Merged pairs stay in the list.** Resolved pairs show a "MERGED" badge rather than disappearing. This gives the analyst a sense of progress and allows undo if needed. An alternative would be to hide them entirely and rely on the progress bar.

## What I'd Explore With More Time

- **Field-level merge controls** — Click to select which value wins per field, with AI pre-selection highlighted
- **Diff highlighting** — Subtle background on rows where Record A ≠ Record B
- **Bulk merge preview** — Before merging 140 Very Low risk pairs at once, show a summary of what will change
- **Undo** — A 10-second undo toast after merge, before the write hits Salesforce
- **Keyboard shortcut bar** — Visible hint showing m=merge, d=dismiss, s=skip, e=escalate for power users
- **Escalation workflow** — What happens after escalate? Assign to a team member, add a note, set a due date
- **Multi-duplicate clusters** — When one account has 5+ matches, show them as a group rather than individual pairs
