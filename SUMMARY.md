# Duplicate Resolution — Design Summary

## Approach

I designed a single-screen Queue Overview that lets a RevOps analyst triage 342 duplicate pairs without navigating between pages. The layout is a split-pane: a scrollable pair list on the left (40%) and a persistent detail view on the right (60%). Clicking a pair in the list instantly shows both records side by side — no modals, no page transitions, no lost context.

This mirrors the mental model of email triage: scan the inbox, review the message, act, move on. For an analyst working through hundreds of pairs in a session, every eliminated click compounds.

## Key Decisions

**Risk tiers over raw confidence scores.** The brief provides AI confidence percentages, but a number like "78%" doesn't tell an analyst what to *do*. I mapped confidence to Kernel's existing risk tier system (Very Low / Low / Medium / High) so the score becomes an action: "Very Low = safe to bulk merge" vs "High = review carefully." The risk score (100 minus confidence) is still shown for precision.

**Split-pane over modal.** The standard pattern is list → click → modal → close → repeat. At 342 pairs that's 684 open/close actions. The persistent detail pane eliminates this entirely. Arrow keys (j/k) let power users work through the queue without touching the mouse.

**Stacked names over inline.** I explored four treatments for showing the pair relationship (inline arrow, stacked, split pill, mini cards). Stacked won because it's the only pattern that scales to multi-duplicate clusters — if one account has 10 matches, they list vertically without breaking the layout.

**Risk chips as the single colour signal.** Early iterations had coloured chips for both risk tiers (green/blue/yellow/red) and classification types (red/blue/purple/amber). Two colour systems competing for attention creates noise. Classification badges became neutral grey — they're categorical labels, not urgency indicators. Risk carries the colour because it drives the action.

**One screen, not a dashboard + detail page.** The brief asks for a Queue Overview. I kept everything in one view: stats bar for the landscape, filters for focus, list for navigation, detail for decision-making. No separate dashboard screen. A RevOps analyst's time is spent resolving pairs, not admiring charts.

## Trade-offs

**No field-level merge picker.** The detail pane shows both records' fields side by side but doesn't yet let the analyst pick which value to keep per field (e.g., keep Record A's address but Record B's owner). This is a natural next step but would add complexity to a single-screen queue view. Kernel's AI recommendation already suggests a primary record, which covers most cases.

**No diff highlighting.** Fields that differ between records aren't visually highlighted. Adding a subtle background or colour to mismatched rows would help the analyst spot decision points faster. Kept it out to avoid scope creep but it's a quick win.

**Fixed risk thresholds.** The tier boundaries (Very Low 0–25, Low 26–60, Medium 61–84, High 85–100) are hardcoded. In production, these should be configurable per-customer, matching Kernel's existing risk tier settings page.

## What I'd Explore With More Time

- **Field-level merge controls** — Click to select which value wins per field, with AI pre-selection highlighted
- **Diff highlighting** — Subtle background on rows where Record A ≠ Record B
- **Bulk merge preview** — Before merging 50 Very Low risk pairs at once, show a summary of what will change
- **Undo** — A 10-second undo toast after merge, before the write hits Salesforce
- **Keyboard shortcut bar** — Visible hint showing m=merge, d=dismiss, s=skip for power users
- **Responsive list density** — Compact mode toggle for analysts who want more pairs visible at once
